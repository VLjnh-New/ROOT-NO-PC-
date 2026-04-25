# Hướng dẫn root Moto G34 5G bằng 2 điện thoại + cáp OTG (không cần PC)

> Phương pháp dùng **Bugjaeger Pro** trên 1 điện thoại Android làm "máy tính" để flash boot patched cho điện thoại Moto G34 5G qua dây cáp OTG.
> Không cần PC, chỉ cần 2 điện thoại + 1 sợi cáp USB-C ↔ USB-C OTG.

---

## Mục lục

- [1. Yêu cầu chuẩn bị](#1-yêu-cầu-chuẩn-bị)
- [2. Tải firmware về máy](#2-tải-firmware-về-máy)
- [3. Unlock bootloader](#3-unlock-bootloader)
- [4. Patch boot.img bằng Magisk](#4-patch-bootimg-bằng-magisk)
- [5. Flash boot patched bằng Bugjaeger](#5-flash-boot-patched-bằng-bugjaeger)
- [6. Setup Magisk chuẩn sau khi root](#6-setup-magisk-chuẩn-sau-khi-root)
- [7. Cài module ẩn root cho app ngân hàng](#7-cài-module-ẩn-root-cho-app-ngân-hàng)
- [8. Kiểm tra root + ẩn root thành công](#8-kiểm-tra-root--ẩn-root-thành-công)
- [9. Các lỗi thường gặp + cách sửa](#9-các-lỗi-thường-gặp--cách-sửa)
- [10. 3 nguyên tắc vàng cần nhớ](#10-3-nguyên-tắc-vàng-cần-nhớ)

---

## 1. Yêu cầu chuẩn bị

### Phần cứng

| Thiết bị | Vai trò |
|---|---|
| **Moto G34 5G** (máy B) | Máy cần root |
| **1 điện thoại Android khác** (máy A, ví dụ Galaxy A26) | Đóng vai "máy tính" chạy Bugjaeger |
| **Cáp OTG USB-C ↔ USB-C** | Nối 2 máy |

### Phần mềm

Cài trên **máy A** (host):
- [Bugjaeger Pro](https://play.google.com/store/apps/details?id=eu.sisik.hackendebug.full) — cầu nối ADB/Fastboot
- [ZArchiver](https://play.google.com/store/apps/details?id=ru.zdevs.zarchiver) — giải nén firmware

Cài trên **máy B** (Moto G34):
- [Magisk APK v30.7+](https://github.com/topjohnwu/Magisk/releases) — patch boot
- ZArchiver — giải nén

### Cảnh báo

- Quá trình này **làm mất bảo hành**.
- Có thể **mất Widevine L1** (Netflix HD bị giảm xuống SD).
- Một số app ngân hàng có thể **không dùng được** sau khi root (cần cài thêm module ẩn root).
- **Backup dữ liệu trước** — quá trình unlock bootloader sẽ wipe hết userdata.

---

## 2. Tải firmware về máy

### 2.1. Xác định version cần tải

Trên Moto G34, vào **Cài đặt → Giới thiệu về điện thoại → Phiên bản phần mềm** → ghi lại chính xác build number.

Ví dụ: `V1UGS35H.75-14-9-3-1-1`

> ⚠️ **Tải đúng firmware match với version đang chạy.** Tải bản cũ hơn sẽ gây bootloop do vbmeta rollback.

### 2.2. Tải firmware từ mirror Lolinet

```
https://mirrors.lolinet.com/firmware/lenomola/2023/fogos/official/RETAPAC/<VERSION>/
```

Thay `<VERSION>` bằng build number của bạn.

### 2.3. Tải bằng curl (có hỗ trợ resume khi mạng rớt)

Trong Termux:

```bash
pkg install curl

curl -L -O -C - "https://mirrors.lolinet.com/firmware/lenomola/2023/fogos/official/RETAPAC/<VERSION>/<TÊN_FILE>.zip"
```

Cờ `-C -` cho phép tiếp tục tải khi mạng đứt — chỉ cần chạy lại lệnh là tải tiếp.

### 2.4. Giải nén firmware

Dùng ZArchiver mở file zip → giải nén ra. Bên trong sẽ có:

- `boot.img` ← **file quan trọng cần patch**
- `vbmeta.img`, `vbmeta_system.img`
- `super.img_sparsechunk.0` đến `.11`
- `vendor_boot.img`, `dtbo.img`, `bootloader.img`, ...

**Backup riêng `boot.img` ra một thư mục dễ tìm** (vd `Download/moto/`) — dùng để patch ở bước sau.

---

## 3. Unlock bootloader

> Bước này chỉ làm 1 lần. Nếu đã unlock rồi, skip qua [Bước 4](#4-patch-bootimg-bằng-magisk).

### 3.1. Bật Developer Options + OEM Unlock

1. Cài đặt → Giới thiệu → bấm **Build number 7 lần** → bật Developer.
2. Cài đặt → Hệ thống → Tùy chọn nhà phát triển → bật:
   - **OEM Unlocking**
   - **USB Debugging**

### 3.2. Lấy unlock code từ Motorola

1. Vào https://en-us.support.motorola.com/app/standalone/bootloader/unlock-your-device-a
2. Đăng ký tài khoản → đăng nhập.
3. Trong Bugjaeger, vào tab Fastboot → gõ:
   ```
   fastboot oem get_unlock_data
   ```
4. Copy chuỗi dài → paste lên website Motorola → nhận **unlock code** qua email.

### 3.3. Unlock

```
fastboot oem unlock <UNLOCK_CODE>
```

Máy sẽ wipe sạch và reboot. Setup lại Android như máy mới.

---

## 4. Patch boot.img bằng Magisk

### 4.1. Cài Magisk APK

Trên Moto, mở trình duyệt → tải `Magisk-v30.7.apk` từ:
https://github.com/topjohnwu/Magisk/releases

Cài file APK → mở app Magisk.

### 4.2. Patch boot.img

1. Trang chủ Magisk → mục **Magisk** (mục trên cùng) → bấm **Cài đặt** (xanh).
2. Cửa sổ Install Options:
   - Bỏ trống tất cả 3 checkbox (Preserve AVB, Preserve force encryption, Recovery mode).
   - Bấm **Next**.
3. Method → chọn **Select and Patch a File**.
4. Bấm **Next** → chọn file **`boot.img`** từ `Download/moto/`.
5. Bấm **LET'S GO** (mũi tên góc phải).
6. Đợi log chạy đến **`- Done!`**.

### 4.3. Lấy file patched

File mới xuất hiện ở `Download/`:
```
magisk_patched-30700_xxxxx.img
```

**Đổi tên ngay** thành `magisk_patched.img` (bỏ dấu cách + `(số)` để tránh lỗi flash):

```
magisk_patched.img
```

### 4.4. Copy file patched sang máy A

Qua Bluetooth, Zalo "Cloud của tôi", hoặc cáp USB.

> ⚠️ **Đừng xóa `boot.img` gốc** — giữ lại để cứu máy nếu bootloop.

---

## 5. Flash boot patched bằng Bugjaeger

### 5.1. Đưa Moto vào Fastboot

1. Tắt nguồn Moto hoàn toàn.
2. Giữ **Nguồn + Volume Down** đến khi vào màn hình Fastboot.
3. Cắm cáp OTG nối **máy A → máy B**.

### 5.2. Mở Bugjaeger trên máy A

1. Cấp quyền USB cho Bugjaeger khi máy A hỏi (chọn làm app mặc định cho thiết bị này).
2. Vào tab **Fastboot** → kiểm tra thấy serial Moto hiện ra:
   ```
   fastboot devices
   ```

### 5.3. Flash boot vào CẢ 2 SLOT

> ⚠️ **Đây là bước quan trọng nhất.** Moto G34 dùng A/B partition. Phải flash cả 2 slot, nếu không sau khi reboot bootloader có thể tự đổi sang slot không có Magisk → mất root.

Trong Bugjaeger Fastboot shell, dùng icon 📎 (kẹp giấy) chọn file `magisk_patched.img` để Bugjaeger tự điền đường dẫn. Sau đó gõ:

```
fastboot --slot=a flash boot /storage/emulated/0/Android/data/eu.sisik.hackendebug.full/cache/cached_fastboot_imgs/magisk_patched.img
```

```
fastboot --slot=b flash boot /storage/emulated/0/Android/data/eu.sisik.hackendebug.full/cache/cached_fastboot_imgs/magisk_patched.img
```

Mỗi lệnh phải báo:
```
Sending 'boot' (98304 KB)    OKAY
Writing 'boot'               OKAY
Finished. Total time: X.XXXs
```

### 5.4. KHÔNG flash vbmeta

> ❌ **TUYỆT ĐỐI KHÔNG flash vbmeta nếu firmware bạn có không trùng version máy.**
> Bootloader đã unlock thì không cần đụng vbmeta. Flash vbmeta cũ sẽ gây rollback v22 vs v28 → bootloop.

### 5.5. Set slot active

```
fastboot set_active a
```

### 5.6. Reboot

```
fastboot reboot
```

### 5.7. Sau khi máy boot xong

- Lần boot đầu lâu hơn bình thường (3–7 phút), kiên nhẫn đợi.
- Có cảnh báo đỏ "Your device is corrupt / Bootloader unlocked" → kệ, kệ máy boot tiếp.
- Mở app Magisk → kiểm tra `Cài đặt: 30.7`, `Ramdisk: Có` ✅

### 5.8. Test reboot lần 2

**Bắt buộc test reboot lần 2** để chắc chắn root ổn định:

1. Khởi động lại máy lần nữa.
2. Mở Magisk → vẫn còn đầy đủ thông tin → root ổn định ✅

Nếu sau reboot lần 2 mà mất tab Magisk → quay lại flash boot vào cả 2 slot (có thể flash chưa xong slot b).

---

## 6. Setup Magisk chuẩn sau khi root

Mở Magisk → biểu tượng ⚙️ ở góc trên phải.

### 6.1. Mục Ứng dụng

| Mục | Cấu hình |
|---|---|
| Chủ đề | Tùy thích |
| Kênh cập nhật | **Ổn định** |
| DNS over HTTPS | **Tắt** |
| Kiểm tra cập nhật | **Bật** ✅ |
| Randomize output name | **Bật** ✅ |
| **Ẩn ứng dụng Magisk** | **TẮT** ❌ |

### 6.2. Mục Magisk

| Mục | Cấu hình |
|---|---|
| Systemless hosts | Tắt (chỉ bật nếu dùng AdAway) |
| **Zygisk** | **BẬT** ✅ |
| **Enforce DenyList** | **BẬT** ✅ |
| Configure DenyList | Tick app cần ẩn root (xem [Bước 7.3](#73-configure-denylist)) |

### 6.3. Mục Superuser

| Mục | Cấu hình |
|---|---|
| Quyền truy cập Superuser | **Chỉ ứng dụng** |
| Chế độ đa người dùng | **Chỉ chủ sở hữu thiết bị** |
| Đáp ứng tự động | **Nhắc nhở** |
| Hết thời gian yêu cầu | **10 giây** |
| Thông báo của Superuser | **Thông báo nổi** |
| **Restrict root capabilities** | **TẮT** ❌ |

### 6.4. Reboot máy 1 lần để áp dụng

---

## 7. Cài module ẩn root cho app ngân hàng

### 7.1. Cài Shamiko (ẩn root toàn diện)

1. Trên Moto, mở trình duyệt → vào:
   https://github.com/LSPosed/LSPosed.github.io/releases
2. Tải file `Shamiko-v1.x.x-release.zip` (mới nhất).
3. Mở Magisk → tab **Mô-đun** → bấm icon ☁️ → chọn file Shamiko zip.
4. Đợi cài xong → bấm **Reboot**.

> Shamiko cần **Enforce DenyList BẬT** để hoạt động. Khi đủ điều kiện, Shamiko sẽ ẩn root toàn diện khỏi mọi app trong DenyList — không cần đổi tên Magisk app.

### 7.2. Cài Play Integrity Fix (vượt qua Play Integrity check)

1. Tải `PlayIntegrityFix_vXX.zip` từ:
   https://github.com/chiteroman/PlayIntegrityFix/releases
2. Cài qua Magisk → Mô-đun → ☁️.
3. Reboot.

### 7.3. Configure DenyList

⚙️ Cài đặt → **Configure DenyList** → tick các app sau (chọn TẤT CẢ process con):

#### Ngân hàng / tài chính Việt Nam
- BIDV SmartBanking, Vietcombank, Vietinbank iPay
- MB Bank, Techcombank, ACB ONE, TPBank, VPBank NEO
- MoMo, ZaloPay, ViettelPay, ShopeePay

#### App chính phủ
- VNeID, VssID

#### Streaming có DRM
- Netflix, HBO Max, Disney+

#### Game chống cheat
- PUBG Mobile, Free Fire, Liên Quân, Genshin Impact

#### Google Services (BẮT BUỘC)
- **Google Play Services** (`com.google.android.gms`) — tick TẤT CẢ process con
- **Google Services Framework**
- **Google Play Store**

→ Reboot 1 lần cuối.

---

## 8. Kiểm tra root + ẩn root thành công

### 8.1. Kiểm tra có root

Cài app **Root Checker** từ Play Store:
- Mở → CHECK ROOT → khi Magisk pop-up xin quyền → **Cấp**.
- Hiện chữ xanh "Congratulations! Root access is properly installed" ✅

### 8.2. Kiểm tra ẩn root thành công

Cài app **YASNAC** từ:
https://github.com/RikkaApps/YASNAC

Mở → Run Test → kết quả mong đợi:
- Basic Integrity: **PASS** ✅
- Device Integrity: **PASS** ✅
- Strong Integrity: **PASS** ✅ (nếu có Play Integrity Fix tốt)

### 8.3. Test app ngân hàng

Mở app ngân hàng → đăng nhập → giao dịch thử → nếu hoạt động bình thường = ẩn root thành công ✅

---

## 9. Các lỗi thường gặp + cách sửa

### Lỗi 1: `cannot load ...: No such file or directory`

**Nguyên nhân:** Tên file có dấu cách hoặc dấu ngoặc đơn `(1)`.

**Cách sửa:** Đổi tên file ra ngắn gọn không có dấu cách:
```
magisk_patched.img
```

### Lỗi 2: `unknown command boot_a` hoặc `unknown command a`

**Nguyên nhân:** Sai cú pháp lệnh fastboot.

**Cách sửa:** Phải có chữ `fastboot` ở đầu lệnh trong Bugjaeger:
```
fastboot --slot=a flash boot /path/to/file.img
fastboot set_active a
fastboot reboot
```

### Lỗi 3: Reboot xong mất tab Superuser / Mô-đun

**Nguyên nhân:**
- Chỉ flash boot vào 1 slot (slot kia là boot gốc, bootloader tự đổi qua).
- Đã bật "Ẩn ứng dụng Magisk".

**Cách sửa:**
1. Vào Fastboot, flash lại boot vào CẢ 2 slot (`--slot=a` và `--slot=b`).
2. Reboot, mở Magisk → KHÔNG bật Hide App nữa.
3. Dùng Shamiko để ẩn root thay vì Hide App.

### Lỗi 4: Bootloop `Can't load Android system. Your data may be corrupt`

**Nguyên nhân:** Flash vbmeta cũ hơn vbmeta hiện tại trên máy → rollback v22 vs v28 → kernel boot được nhưng system reject.

**Cách sửa:**
1. Vào Fastboot.
2. Flash lại boot **chưa patched** (boot.img gốc) vào cả 2 slot.
3. Nếu vẫn bootloop → phải tải đúng firmware match version máy hiện tại, flash full firmware (super.img + boot + vbmeta + vendor_boot + dtbo) cho đồng bộ.
4. Phương án cuối: Factory data reset trong recovery — sẽ mất hết dữ liệu nhưng đôi khi cứu được.

### Lỗi 5: `WARNING: vbmeta_a anti rollback downgrade, 22 vs 28`

**Nguyên nhân:** Firmware bạn dùng cũ hơn ROM máy đang chạy.

**Cách sửa:**
- **Nguyên tắc:** Đừng flash vbmeta cũ. Bootloader đã unlock thì không cần đụng vbmeta.
- Nếu đã lỡ flash → phải tải firmware đúng version (mới hơn) rồi flash lại.

### Lỗi 6: Hộp thoại "Khôi phục ứng dụng Magisk"

**Nguyên nhân:** App vỏ bọc (sau khi bật Hide App) không xin được root từ daemon.

**Cách sửa:**
- Bấm **CÀI ĐẶT** → app sẽ tự khôi phục về bản gốc.
- Sau đó **ĐỪNG bật Hide App nữa** — dùng Shamiko để ẩn root.

---

## 10. 3 nguyên tắc vàng cần nhớ

### ✅ 1. Flash boot vào CẢ 2 SLOT
```
fastboot --slot=a flash boot magisk_patched.img
fastboot --slot=b flash boot magisk_patched.img
```
Moto là A/B device, chỉ flash 1 slot sẽ bị mất root sau reboot.

### ❌ 2. KHÔNG flash vbmeta
Bootloader đã unlock rồi, không cần đụng vbmeta. Flash vbmeta cũ → bootloop chắc chắn.

### ❌ 3. KHÔNG bật "Ẩn ứng dụng Magisk"
Đây là tính năng gây mất tab Magisk + lỗi daemon. Dùng **Shamiko + DenyList** để ẩn root cho app ngân hàng — an toàn, hiệu quả hơn nhiều.

---

## Liên kết hữu ích

- Magisk: https://github.com/topjohnwu/Magisk
- Shamiko: https://github.com/LSPosed/LSPosed.github.io/releases
- Play Integrity Fix: https://github.com/chiteroman/PlayIntegrityFix
- YASNAC (kiểm tra ẩn root): https://github.com/RikkaApps/YASNAC
- Mirror firmware Moto: https://mirrors.lolinet.com/firmware/lenomola/
- Bugjaeger Pro: https://play.google.com/store/apps/details?id=eu.sisik.hackendebug.full

---

## License

Hướng dẫn này được viết dựa trên kinh nghiệm thực tế root Moto G34 5G bằng 2 điện thoại + cáp OTG. Sử dụng theo License MIT — bạn được tự do copy, chỉnh sửa, chia sẻ.

**Lưu ý:** Root sẽ làm mất bảo hành. Tác giả không chịu trách nhiệm nếu thiết bị của bạn gặp vấn đề. Làm theo hướng dẫn với rủi ro của riêng bạn.
