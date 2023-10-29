# WZR-HP-G300NH-JP-
HUONG DAN NANG CAP - UNBRICK MODEM Buffalo WZR-HP-G300NH (RTL8366RB switch) (Japanese version) / Firmware setup for Buffalo WZR-HP-G300NH (JP) using serial interface

I. UPDATE TO FIRMWARE DD-WRT:
Nếu Router đang dùng Firmware DD-WRT chỉ cần tải bản nâng cấp:
http://download1.dd-wrt.com/dd-wrtv2/downloads/betas/2020/
Nhớ chọn bản wzr-hp-g300nh-dd-wrt-webupgrade-MULTI.bin
Khi nâng cấp xong, reboot lại modem.
	
II. UNBRICK FROM DD-WRT:
1. Đưa Router từ DD-WRT VỀ FIRMWARE GỐC (tftp64):
   Kết nối Router vào máy tính dùng giao thức TTL/Serial (pinout.jpg) rồi dùng trình tftp để cài đặt, Sử dụng trình Tera Term VT
   Tham khao: https://scarygliders.net/2010/02/23/hacking-around-the-japanese-buffalo-wzr-hp-g300n/
   Kêt nối dây LAN từ máy tính vô eth0 Router, vào Network set IP máy tính là 192.168.11.2, Server IP (router) là 192.168.11.1.
~~~
OUTDATE: Mở trình tftp64, set Current Dir: ../Dowloads/wzrg300nh_original.bin (nơi chứa file tftp để cài Router ) (wzrg300nh_original.bin: Its a stock 1.65 firmware and has been modified to work with DD-WRT web upgrade interface -- http://g300nh.blogspot.com/2010/06/flash-between-dd-wrt-and-official.html?m=1)
~~~
Trước hết phải đưa Router về Firmware gốc, -> wzrg300nh_original.bin
LƯU Ý: Bản 165 up lên được nhưng không up lên tiếp DD-WRT được. Hai bản này đều có Magic number ở đầu file (sử dụng Hex Editor kiểm tra). Nên nhớ chỉ up firmware bằng tftp có Magic Number, nếu không thì tiến trình đưa Router về Firmware gốc sẽ bị lỗi.

2. Dùng TeraTerm VT flash firmware:
   Chạy trình tftp (TeraTerm VT) serial port setup: Port: COM 4, Speed: 115200, 8, N, 1.
   Khi hiện dòng RTL8366RB nhấn ngay Ctr-C > any key.
   nhập lệnh:
   > ar7100> printenv //(xem thông số MODEM)
   
~~~	
bootargs=console=ttyS0,115200 root=31:03 rootfstype=jffs2 init=/sbin/init mtdparts=ar9100-nor0:256k(u-boot),128k(u-boot-env),1024k(uImage),31104k(rootfs),128k@32640k(ART),128k@32512k(properties)
bootcmd=bootm 0xbe060000
bootdelay=4
baudrate=115200
ethaddr=02:AA:BB:CC:DD:1A
tmp_ram=81F00000
tmp_bottom=83F00000
fw_eaddr=BE060000 BFFDFFFF
uboot_eaddr=BE000000 BE03FFFF
u_fw=erase $fw_eaddr; cp.b $fileaddr BE060000 $filesize; bootm BE060000;
ut_fw=tftp $tmp_ram firmware.bin; erase $fw_eaddr; cp.b $fileaddr BE060000 $filesize; bootm BE060000;
ut_uboot=tftp $tmp_ram u-boot.bin; protect off $uboot_eaddr; erase $uboot_eaddr; cp.b $fileaddr BE000000 $filesize;
melco_id=RD_BB08009
tftp_wait=4
uboot_ethaddr=02:AA:BB:CC:DD:1A
DEF-p_wireless_ath0_11bg-authmode=psk
DEF-p_wireless_ath0_11bg-crypto=tkip+aes
DEF-p_wireless_ath0_11bg-authmode_ex=mixed-psk
buf_ver=1.06
build_date=Sep  2 2009 - 08:20:59
buf_crc=09E8C95B
hw_rev=0
pincode=32113808
custom_id=0
DEF-p_wireless_ath0_11bg-wpapsk=erc8wgs898pmb
fileaddr=81F00000
ipaddr=192.168.11.1
serverip=192.168.11.2
accept_open_rt_fmt=1
filesize=1A6A1FC
region=EU
stdin=serial
stdout=serial
stderr=serial
loadaddr=81F00000
ethact=eth1

Environment size: 1176/131068 bytes
~~~
  nhập lệnh: 
> ar7100> setenv region eu/jp //(đổi vùng cho router, không quan trong, cứ để vùng là JP cũng được)
> ar7100> saveenv //(lưu lại)
Copy my-tftp.bin vào bộ nhớ tạm:
> ar7100> loady //Chọn File>Transfer>YMODEM>Send (chọn file my-tftp.bin)
hoặc dùng lệnh :
> ar7100> tftpboot 81f00000 my-tftp.bin
LƯU Ý: Các file tftp này phải có Magic number mới có thể cài được vào WZR-HP-G300NH bản Nhật, Tên tệp nên rename ngắn lại.
> ar7100> iminfo //kiem tra file da tai len, Neu Bad Header Checksum > thi image nay khong dung duoc.
~~~
## Checking Image at 81f00000 …
Image Name:   MIPS X-WRT Linux-5.15.135
   Created:      2023-10-21  11:21:24 UTC
   Image Type:   MIPS Linux Kernel Image (lzma compressed)
   Data Size:    2356322 Bytes =  2.2 MB
   Load Address: 80060000
   Entry Point:  80060000
   Verifying Checksum ... OK
~~~	
LƯU Ý : To summarise, basically, if you want the Buffalo Japan-provided u-boot loader to recognise your firmware image of choice, the firmware image needs to start with the magic number sequence mentioned above in order for the firmware image to be recognised. This works for that dd-wrt image AND also the OpenWRT images they’re now producing for the WZR-HP-G300NH.
	
Khi up lên xong !!!NHỚ!!! để ý dung lượng Byte đã upload dạng HEX
Xóa fimware củ:
> ar7100> erase BE060000 BFFDFFFF	//PHÂN VÙNG FIRMWARE CHÍNH
Khi  xong, Copy Firmware mới từ bộ nhớ tạm qua:
> ar7100> cp.b 81f00000 be060000 a64000 (LƯU Ý: ở đây, a64000 tương ứng 10895360 bytes, chỉ là ví dụ, nhớ thay đổi dung lượng firmware mới dạng Hex ở bộ nhớ tạm vừa up lên)
> ar7100> bootm BE060000 //(để chạy Firmware mới).
	
2. Nâng cấp tiếp lên DD-WRT:
Vào Web: http://download1.dd-wrt.com/dd-wrtv2/downloads/betas/2020/ tải bản mới nhất/ hoặc ổn định nhất, chỉ tải bản buffalo_to_ddwrt_webflash-MULTI.bin -> tiến hành quá trình nâng cấp bình thường qua trang Web.
Xong
	
III. Nâng cấp to OpenWrt/X-WRT
Cách up tương tự như trên, tải tftp image file: Buffalo WZR-HP-G300NH (RTL8366RB/RTL8366S switch) tftp.bin từ https://downloads.x-wrt.com/rom/
Mở file trong HxD, xóa block ở đầu file cho đến Magic number thì dừng (27 05 19 56), lần sau khi nâng cấp trong Web thì chỉ cần cài bản x-wrt-xx.xx-bxxxxxxxxxxxx-ath79-generic-buffalo_wzr-hp-g300nh-rb-squashfs-sysupgrade.bin

IV. Useful Links

https://scarygliders.net/2010/02/23/hacking-around-the-japanese-buffalo-wzr-hp-g300n/
http://g300nh.blogspot.com/2010/06/firmware-flash-and-brick-recovery.html
https://dd-wrt.com/support/router-database/
https://digi9.net/221/openwrt-fix-loi-firmware-router-buffalo-wzr-hp-g450h/
http://g300nh.blogspot.com/2010/06/flash-between-dd-wrt-and-official.html
