在 Ubuntu 中，您可以使用 `lsof` 或 `netstat` 命令来查找占用特定端口的程序。使用 ==[sudo lsof -i:端口號](https://www.google.com/search?q=sudo+lsof+-i%3A%E7%AB%AF%E5%8F%A3%E8%99%9F&oq=ubunut+%E6%9F%A5%E5%87%BA%E4%BD%94%E7%94%A8port&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIKCAEQABiiBBiJBTIKCAIQABiiBBiJBTIHCAMQABjvBTIHCAQQABjvBdIBCDgzODNqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8&mstk=AUtExfCvotdxvrqJotZ7Rs17cxaWIi5iV349s-cM0gDw22Txr1_ky4ZWMjbTaZujNNuo9qcPRWIyD6riobUj_8IwyM9-fx8rl0cNZFOu9M3Dya8cFZ6w2PRnWCxXWG6heU7gUAyBlfQ_edhpFCFoLGImZoHpKCwaaQsVnCRIyO32SCFl3K5zfx7T4SYHP7Kh8lfA3_YLcpfIee11bG807K3P1x-E3q9SjFD0N8WSflzKDQDEKLSYhXWB3uSSLwYr9AaxspipMefN8ZDtGj-y_mJP7pRm&csui=3&ved=2ahUKEwjQnb632NWQAxXre_UHHXX0Hh8QgK4QegQIARAC)== 可以快速找出哪個程序正在使用該端口，而 [netstat -anp | grep 端口號](https://www.google.com/search?q=netstat+-anp+%7C+grep+%E7%AB%AF%E5%8F%A3%E8%99%9F&oq=ubunut+%E6%9F%A5%E5%87%BA%E4%BD%94%E7%94%A8port&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIKCAEQABiiBBiJBTIKCAIQABiiBBiJBTIHCAMQABjvBTIHCAQQABjvBdIBCDgzODNqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8&mstk=AUtExfCvotdxvrqJotZ7Rs17cxaWIi5iV349s-cM0gDw22Txr1_ky4ZWMjbTaZujNNuo9qcPRWIyD6riobUj_8IwyM9-fx8rl0cNZFOu9M3Dya8cFZ6w2PRnWCxXWG6heU7gUAyBlfQ_edhpFCFoLGImZoHpKCwaaQsVnCRIyO32SCFl3K5zfx7T4SYHP7Kh8lfA3_YLcpfIee11bG807K3P1x-E3q9SjFD0N8WSflzKDQDEKLSYhXWB3uSSLwYr9AaxspipMefN8ZDtGj-y_mJP7pRm&csui=3&ved=2ahUKEwjQnb632NWQAxXre_UHHXX0Hh8QgK4QegQIARAD) 則能列出所有狀態的連接，並顯示程式PID。 

使用 `lsof`

`lsof` (list open files) 命令能夠顯示系統中所有被打開的檔案，包括網絡連接。 

- **查看特定端口**
    - 執行命令：`sudo lsof -i :端口號`
    - 例如，要查看80端口的占用情況，請輸入：`sudo lsof -i :80`
    - 這將會列出所有使用該端口的進程信息，包括進程ID（PID）和進程名稱。 

使用 `netstat`

`netstat` 命令可以顯示網路連接、路由表和網路接口的統計資訊。 

- **查看所有監聽端口**
    - 執行命令：`netstat -tuln`
    - `-t` 顯示TCP連接，`-u` 顯示UDP連接，`-l` 只顯示監聽中的端口，`-n` 以數字形式顯示地址和端口。
- **查看特定端口的狀態**
    - 執行命令：`netstat -anp | grep 端口號`
    - 例如，要查看3306端口的狀態，請輸入：`netstat -anp | grep 3306`。
    - 在結果中，如果端口旁邊顯示 `LISTEN`，則表示該端口已被占用。 

如何終止進程

找到佔用端口的進程後，您可以根據其PID終止該進程。 

- 執行命令：`sudo kill 进程ID`
- 您也可以使用 `kill -9 进程ID` 來強制終止進程。