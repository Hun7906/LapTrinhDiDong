# BÁO CÁO THỰC HÀNH ANDROID BASICS COMPOSE – UNIT 8, PATHWAY 2
## DỰ ÁN: JUICE TRACKER – VIEWS IN COMPOSE

---

## 1. Giới thiệu mục tiêu bài thực hành

Bài thực hành **Android Basics with Compose – Unit 8, Pathway 2 (Views in Compose)** tập trung vào khả năng tương tác giữa hệ thống giao diện Android View truyền thống và Jetpack Compose thông qua API `AndroidView`.

Mục tiêu cụ thể của dự án **Juice Tracker**:
- Kiểm tra dự án hiện có tại `D:\Lab12_ltdd` (là dự án mẫu `com.example.lab12_ltdd` chưa chứa mã nguồn Juice Tracker) và clone mã nguồn khởi đầu chính thức từ Google nhánh `compose-starter` vào thư mục riêng `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker` mà không ghi đè dự án gốc.
- Nhúng 3 thành phần Android View truyền thống vào giao diện Compose hiện có của ứng dụng Juice Tracker:
  1. **`Spinner`**: Chọn màu sắc nước ép từ enum `JuiceColor`.
  2. **`RatingBar`**: Chọn số sao đánh giá nguyên từ 1 đến 5.
  3. **`AdView`**: Hiển thị biểu ngữ quảng cáo thử nghiệm Google Mobile Ads ở phía trên danh sách nước ép.
- Đảm bảo luồng dữ liệu hai chiều ổn định giữa Android View và trạng thái Compose (`JuiceTrackerViewModel` + Room Database), giữ nguyên toàn bộ chức năng thêm, sửa, xóa nước ép sẵn có và biên dịch thành công bằng Gradle (`assembleDebug`).

### Thông tin dự án Juice Tracker (nhánh `compose-starter`)
- **Đường dẫn dự án**: `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker`
- **Package name / Namespace**: `com.example.juicetracker`
- **minSdk**: `24`
- **compileSdk / targetSdk**: `35`
- **Kotlin / Compose Compiler plugin**: `2.1.0`
- **Android Gradle Plugin (AGP)**: `8.8.0`
- **Gradle Wrapper**: `8.12`
- **Compose BOM**: `2024.12.01`

---

## 2. Các kiến thức Android View Interoperability

Trong thực tế phát triển ứng dụng Android bằng Jetpack Compose, nhiều thành phần UI truyền thống hoặc SDK bên thứ ba (như `AdView`, `WebView`, `MapView`, `Spinner`, `RatingBar`) vẫn được xây dựng dựa trên hệ thống `android.view.View`. Jetpack Compose cung cấp cơ chế **View Interoperability** với các nguyên tắc cốt lõi:

- **API `AndroidView`**: Composable cầu nối cho phép khởi tạo, hiển thị, cập nhật và giải phóng một `View` truyền thống bên trong cây giao diện Compose.
- **Unidirectional Data Flow (UDF)**: Trạng thái vẫn được quản lý tập trung ở Compose / `ViewModel` (`StateFlow<Juice>`). Khi trạng thái thay đổi, Compose truyền dữ liệu xuống Android View qua khối `update`. Khi người dùng tương tác với Android View, listener truyền sự kiện ngược về Compose qua lambda callback (ví dụ `onColorChange`, `onRatingChange`).
- **Quản lý Recomposition và vòng đời**:
  - Các tác vụ chỉ nên thực hiện một lần (như khởi tạo `View`, gán `ArrayAdapter`, cấu hình `stepSize`, tải quảng cáo `loadAd()`) được đặt trong `factory` để tránh lặp lại khi recomposition.
  - Tài nguyên nặng hoặc có vòng đời riêng (như `AdView`) cần được đồng bộ với `Lifecycle` (`resume()`, `pause()`) và giải phóng (`destroy()`) trong `onRelease`.

---

## 3. Danh sách file đã tạo và chỉnh sửa

### Các file tạo mới
1. `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\src\main\java\com\example\juicetracker\ui\bottomsheet\ColorSpinnerRow.kt`
   - Chứa lớp `SpinnerAdapter` và Composable `ColorSpinnerRow`.
2. `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\src\main\java\com\example\juicetracker\ui\bottomsheet\RatingInputRow.kt`
   - Chứa Composable `RatingInputRow`.
3. `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\src\main\java\com\example\juicetracker\ui\homescreen\AdBanner.kt`
   - Chứa Composable `AdBanner`.
4. `D:\Lab12_ltdd\UNIT8_PATHWAY2_REPORT.md`
   - Báo cáo kết quả thực hành.

### Các file đã chỉnh sửa
1. `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\src\main\java\com\example\juicetracker\ui\bottomsheet\EntryBottomSheet.kt`
   - Tích hợp `ColorSpinnerRow` và `RatingInputRow` vào `SheetForm` (sau phần nhập Description, trước `ButtonRow`).
   - Bổ sung hàm tiện ích `findColorIndex(color: String): Int`.
2. `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\src\main\java\com\example\juicetracker\ui\JuiceTrackerApp.kt`
   - Tích hợp `AdBanner` phía trên `JuiceTrackerList` trong `Column(Modifier.padding(contentPadding))` với `Modifier.fillMaxWidth()` và padding trên/dưới bằng `dimensionResource`.
3. `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\build.gradle.kts`
   - Thêm thư viện `implementation("com.google.android.gms:play-services-ads:22.2.0")`.
4. `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\src\main\AndroidManifest.xml`
   - Thêm quyền `android.permission.INTERNET` và thẻ `<meta-data>` cấu hình `com.google.android.gms.ads.APPLICATION_ID` thử nghiệm.

*(Ghi chú: File `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\local.properties` được sao chép từ dự án gốc để Gradle trỏ đúng `sdk.dir=D:\Android`; file này nằm trong `.gitignore`.)*

---

## 4. Giải thích chức năng Spinner

- **File cài đặt**: `app/src/main/java/com/example/juicetracker/ui/bottomsheet/ColorSpinnerRow.kt`
- **File tích hợp**: `app/src/main/java/com/example/juicetracker/ui/bottomsheet/EntryBottomSheet.kt`

Cơ chế hoạt động:
- Lớp `SpinnerAdapter(val onColorChange: (Int) -> Unit)` kế thừa `AdapterView.OnItemSelectedListener`:
  - `onItemSelected(...)`: Gọi `onColorChange(position)` để gửi chỉ số màu được chọn về Compose.
  - `onNothingSelected(...)`: Mặc định gọi `onColorChange(0)` (màu đầu tiên trong `JuiceColor`).
- Composable `ColorSpinnerRow(colorSpinnerPosition, onColorChange, modifier)`:
  - Chuyển đổi toàn bộ giá trị trong enum `JuiceColor` sang danh sách tên chuỗi đã bản địa hóa bằng `stringResource(juiceColor.label)`.
  - Đặt nhãn `"Color"` (`R.string.color`) thông qua `InputRow`.
  - Nhúng `android.widget.Spinner` qua `AndroidView(modifier = Modifier.fillMaxWidth(), ...)`:
    - Trong `factory`: Khởi tạo `Spinner` và gán `ArrayAdapter` sử dụng layout chuẩn `android.R.layout.simple_spinner_dropdown_item`.
    - Trong `update`: Tạm xóa listener cũ (`spinner.onItemSelectedListener = null`), đồng bộ vị trí hiện tại bằng `spinner.setSelection(colorSpinnerPosition)`, sau đó gán `SpinnerAdapter(onColorChange)` mới để luôn giữ closure mới nhất của `juice` và tránh ghi đè dữ liệu cũ khi chuyển đổi giữa các mục nước ép.
- Hàm `findColorIndex(color: String): Int` trong `EntryBottomSheet.kt` ánh xạ tên chuỗi của màu (`Juice.color`) sang chỉ số tương ứng trong `JuiceColor.values()`.
- Trong `SheetForm`, khi người dùng chọn màu mới, callback cập nhật `onUpdateJuice(juice.copy(color = JuiceColor.values()[color].name))`.

---

## 5. Giải thích chức năng RatingBar

- **File cài đặt**: `app/src/main/java/com/example/juicetracker/ui/bottomsheet/RatingInputRow.kt`
- **File tích hợp**: `app/src/main/java/com/example/juicetracker/ui/bottomsheet/EntryBottomSheet.kt`

Cơ chế hoạt động:
- Composable `RatingInputRow(rating: Int, onRatingChange: (Int) -> Unit, modifier: Modifier = Modifier)` sử dụng `InputRow` với nhãn `"Rating"` (`R.string.rating`).
- Bên trong `AndroidView`:
  - `factory`: Khởi tạo `android.widget.RatingBar(context)` và đặt `stepSize = 1f` để chỉ nhận giá trị số sao nguyên.
  - `update`: Gỡ tạm listener (`setOnRatingBarChangeListener(null)`), đồng bộ giá trị từ Compose xuống bằng `ratingBar.rating = rating.toFloat()`, rồi đăng ký lại `setOnRatingBarChangeListener { _, newRating, _ -> onRatingChange(newRating.toInt()) }`. Cách làm này ngăn callback cũ kích hoạt ngoài ý muốn khi mở chỉnh sửa một nước ép khác và đảm bảo hiển thị đúng số sao đã lưu.
- Trong `SheetForm`, `RatingInputRow` được đặt ngay sau `ColorSpinnerRow` và trước `ButtonRow`, cập nhật trạng thái bằng `onUpdateJuice(juice.copy(rating = rating))`.

---

## 6. Giải thích chức năng AdView

- **File cài đặt**: `app/src/main/java/com/example/juicetracker/ui/homescreen/AdBanner.kt`
- **File tích hợp**: `app/src/main/java/com/example/juicetracker/ui/JuiceTrackerApp.kt`

Cơ chế hoạt động:
- Composable `AdBanner(modifier: Modifier = Modifier)` nhúng `com.google.android.gms.ads.AdView` thông qua `AndroidView`.
- Trong `factory` (chỉ chạy một lần khi tạo View, không chạy lại khi Compose recomposition):
  - Đặt kích thước `setAdSize(AdSize.BANNER)`.
  - Gán mã đơn vị quảng cáo thử nghiệm chính thức của Google: `ca-app-pub-3940256099942544/6300978111`.
  - Tạo yêu cầu `AdRequest.Builder().build()` và gọi `loadAd(...)`.
- Xử lý vòng đời:
  - Sử dụng `DisposableEffect` kết hợp `LocalLifecycleOwner` và `LifecycleEventObserver` để gọi `adView.resume()` khi `ON_RESUME` và `adView.pause()` khi `ON_PAUSE`.
  - Sử dụng tham số `onRelease` của `AndroidView` để gọi `adView.destroy()` khi `AdBanner` rời khỏi cây Composition / màn hình bị hủy.
- Trong `JuiceTrackerApp.kt`, `AdBanner` được đặt ngay phía trên `JuiceTrackerList` bên trong `Column(Modifier.padding(contentPadding))` với `Modifier.fillMaxWidth().padding(top = dimensionResource(R.dimen.padding_medium), bottom = dimensionResource(R.dimen.padding_small))`, giúp banner nằm gọn trên đầu danh sách và không che khuất các mục nước ép.

---

## 7. Vai trò của `AndroidView`, `factory` và `update`

- **`AndroidView`**: Là Composable chịu trách nhiệm tạo một `AndroidViewHolder` để đo đạc (measure), bố trí (layout), vẽ (draw) và chuyển tiếp sự kiện chạm giữa cây Compose và đối tượng `android.view.View`.
- **`factory: (Context) -> T`**:
  - Được gọi **một lần duy nhất** khi Composable đi vào Composition lần đầu.
  - Dùng để khởi tạo `View` (`Spinner`, `RatingBar`, `AdView`) và thiết lập các thuộc tính tĩnh hoặc tác vụ chỉ nên chạy một lần (`ArrayAdapter`, `stepSize = 1f`, `setAdSize`, `adUnitId`, `loadAd`).
  - Việc đặt `loadAd()` trong `factory` thay vì `update` giúp tránh gửi lặp yêu cầu quảng cáo mỗi lần Compose recompose.
- **`update: (T) -> Unit`**:
  - Được gọi ngay sau `factory` và mỗi khi trạng thái Compose đọc bên trong nó thay đổi (recomposition).
  - Dùng để đồng bộ trạng thái từ Compose xuống `View` (`spinner.setSelection(...)`, `ratingBar.rating = ...`) và gắn listener mới tham chiếu tới callback hiện hành.
- **`onRelease: (T) -> Unit`**:
  - Được gọi khi `AndroidView` rời khỏi Composition, dùng để giải phóng tài nguyên như `adView.destroy()`.

---

## 8. Cấu hình Gradle và AndroidManifest

### `app/build.gradle.kts`
Đã bổ sung dependency Google Mobile Ads theo đúng hướng dẫn Code Lab mà không làm xung đột với AGP `8.8.0`, Kotlin `2.1.0` hay các dependency Room/Compose hiện có:

```kotlin
implementation("com.google.android.gms:play-services-ads:22.2.0")
```

### `app/src/main/AndroidManifest.xml`
- Bổ sung quyền truy cập mạng để tải quảng cáo thử nghiệm:
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
- Bổ sung metadata App ID thử nghiệm chính thức của Google bên trong thẻ `<application>`:
```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-3940256099942544~3347511713" />
```

---

## 9. Kết quả kiểm tra build

Đã thực hiện kiểm tra trực tiếp trên Windows trong thư mục dự án `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker`:

1. **Lệnh `.\gradlew.bat assembleDebug --stacktrace`**:
   - **Kết quả**: `BUILD SUCCESSFUL in 3m 37s` (38 actionable tasks executed).
   - Gradle tự động tải và cài đặt Android SDK Build-Tools 35 (`35.0.0`) và Android SDK Platform 35 vào `D:\Android`.
   - `kspDebugKotlin`, `compileDebugKotlin`, `processDebugMainManifest`, `packageDebug` đều thành công, **không có lỗi biên dịch** và **không có cảnh báo mới** trong các file vừa tạo/chỉnh sửa.
   - File APK đầu ra: `D:\Lab12_ltdd\basic-android-kotlin-compose-training-juice-tracker\app\build\outputs\apk\debug\app-debug.apk` (15,855,005 bytes).

2. **Lệnh `.\gradlew.bat testDebugUnitTest`**:
   - **Kết quả**: `BUILD SUCCESSFUL in 14s` (`NO-SOURCE` vì nhánh `compose-starter` gốc không chứa bộ unit test sẵn).

---

## 10. Những chức năng đã kiểm thử và những phần chưa thể xác minh

### Những chức năng đã kiểm thử thực tế trên thiết bị thật (`adb` – thiết bị `d5947d19`)
- **Cài đặt và khởi chạy ứng dụng**: `adb install -r app-debug.apk` thành công (`Success`), ứng dụng khởi chạy ổn định (`MainActivity`), kiểm tra `logcat` không ghi nhận lỗi crash (`AndroidRuntime:E`).
- **TEST 3 – AdBanner**: Đã xác minh qua ảnh chụp màn hình thiết bị thật — biểu ngữ quảng cáo thử nghiệm của Google (`"Quảng cáo thử nghiệm – Nice job! This is a 468x60 test ad."`) hiển thị rõ ràng phía trên danh sách nước ép, có khoảng đệm trên/dưới hợp lý và không che khuất nội dung danh sách.
- **TEST 2 – RatingBar & Thêm nước ép**: Đã xác minh trên thiết bị thật — mở biểu mẫu `EntryBottomSheet` (có đầy đủ các hàng `Juice Name`, `Description`, `Color`, `Rating`, nút `CANCEL` và `SAVE`), chọn 4 sao thay cho mặc định 3 sao và lưu thành công mục nước ép (`"cam ép"`, `"ngon"`, hiển thị đủ 4 sao trên danh sách).

### Những phần chưa được kiểm thử tự động toàn bộ
- **TEST 1 – Kiểm thử đầy đủ cả 7 màu trong Spinner** (chọn lần lượt Red, Blue, Green, Cyan, Yellow, Magenta, Orange, lưu rồi mở lại biểu mẫu kiểm tra đồng bộ từng màu): Mới xác minh cấu trúc hiển thị `Color` + logic đồng bộ `setSelection(colorSpinnerPosition)` / `findColorIndex()`, chưa chạy kịch bản chọn hết toàn bộ 7 màu tự động trên thiết bị để tránh can thiệp thao tác thủ công của người dùng trên điện thoại thật.
- **TEST 4 – Sửa và xóa nước ép hiện có trên thiết bị**: Chức năng thêm mới đã hoạt động; chức năng chỉnh sửa và xóa được giữ nguyên kết nối `onUpdate` / `onDelete` với `JuiceTrackerViewModel` và `RoomJuiceRepository`, cần người dùng thao tác kiểm chứng thêm trên thiết bị để xác nhận đầy đủ toàn bộ kịch bản.
- **Unit Test / Instrumented UI Test tự động**: Dự án `compose-starter` mặc định chưa kèm thư mục `test`/`androidTest`, nên mới kiểm chứng bằng biên dịch Gradle và chạy trực tiếp APK trên thiết bị.
