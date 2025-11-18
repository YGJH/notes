## 摘要

要使用 **exe4j** 打包 Java 應用為 Windows 可執行檔（.exe），首先必須將專案編譯並打包成含有 Main-Class 設定的可執行 JAR 檔，接著在 **exe4j Wizard** 中選擇「JAR in EXE」模式，指定剛才產出的 JAR 檔及主類別，並設定捆綁 JRE 與圖示等參數，最後生成 .exe 檔。以下將分為兩大步驟：一、如何從原始碼產生可執行 JAR；二、如何在 exe4j 中進行配置與打包。

---

## 一、從專案產生可執行 JAR

### 1. 編譯 Java 類別

在命令列中切換到您的專案目錄，並執行：

```bash
javac -d out/production/MyApp src/com/example/*.java
```

此命令會將 `.java` 檔編譯後輸出至 `out/production/MyApp` 資料夾 ([TomTom Engineering Blog](https://engineering.tomtom.com/create-java-jar/?utm_source=chatgpt.com "Back to Basics: How to create a java JAR using command line"))。

### 2. 建立 Manifest 檔

在專案根目錄下，建立 `META-INF/MANIFEST.MF`，內容例如：

```text
Manifest-Version: 1.0
Main-Class: com.example.MainApp
```

`Main-Class` 必須對應到含有 `public static void main` 方法的類別，全名需包含 package 路徑 ([TomTom Engineering Blog](https://engineering.tomtom.com/create-java-jar/?utm_source=chatgpt.com "Back to Basics: How to create a java JAR using command line"))。

### 3. 使用 jar 指令打包

執行以下命令以建立可執行 JAR：

```bash
jar cfm MyApp.jar META-INF/MANIFEST.MF -C out/production/MyApp .
```

- `c`：create，建立新檔案
    
- `f`：file，輸出至檔案
    
- `m`：manifest，使用自訂 Manifest ([Oracle 文檔](https://docs.oracle.com/javase/tutorial/deployment/jar/build.html?utm_source=chatgpt.com "Creating a JAR File (The Java™ Tutorials > Deployment ..."))。
    

執行成功後，可透過 `java -jar MyApp.jar` 測試執行效果 ([Baeldung](https://www.baeldung.com/java-create-jar?utm_source=chatgpt.com "Guide to Creating and Running a Jar File in Java | Baeldung"))。

### 4. （可選）使用 IDE 快速導出

若使用 Eclipse，可右鍵點擊專案 → **Export…** → **Runnable JAR file**，設定 Launch configuration、Library handling、輸出路徑等，快速產生可執行 JAR ([Stack Overflow](https://stackoverflow.com/questions/4597866/java-creating-jar-file?utm_source=chatgpt.com "Java creating .jar file - Stack Overflow"))。

---

## 二、在 exe4j 中配置並打包 .exe

### 1. 啟動 exe4j Wizard

打開 **exe4j**，在第一步選擇 **“JAR in EXE”** 模式，此模式會將指定的 JAR 內嵌到可執行檔中，產生單一 .exe 分發檔案 ([ej-technologies](https://www.ej-technologies.com/resources/exe4j/help/doc/wizard.html?utm_source=chatgpt.com "exe4j Help - Configuration wizard - ej-technologies"))。

### 2. 設定 Java Invocation

- **Main class**：輸入 `com.example.MainApp`（對應 Manifest 中的 Main-Class）。
    
- **Classpath**：新增剛打包的 `MyApp.jar`；若有其他依賴 JAR，也一併加入 ([ej-technologies](https://www.ej-technologies.com/resources/exe4j/help/doc/help.pdf?utm_source=chatgpt.com "[PDF] exe4j Manual - ej-technologies"))。
    
- **Bundled JRE**：可選擇將特定版本的 JRE 映像放入可執行檔，確保目標機器無需預先安裝 Java ([bockytech.com.tw](https://www.bockytech.com.tw/PDF-File/exe4jManual.pdf?utm_source=chatgpt.com "[PDF] exe4j Manual"))。
    

### 3. 配置可執行檔資訊

- **Executable info**：設定應用名稱、版本、圖示（.ico）等基本資訊。
    
- **Single instance mode**（如需單例執行）：可設定允許僅執行一個實例及啟動事件監聽 ([ej-technologies](https://www.ej-technologies.com/resources/exe4j/help/doc/help.pdf?utm_source=chatgpt.com "[PDF] exe4j Manual - ej-technologies"))。
    

### 4. 生成與測試

完成 Wizard 向導後，點擊 **“Generate EXE”**，或使用命令行方式：

```bash
exe4jc MyApp.exe4j
```

此命令會根據 `MyApp.exe4j` 配置檔生成最終的 `MyApp.exe` ([ej-technologies](https://www.ej-technologies.com/resources/exe4j/help/doc/help.pdf?utm_source=chatgpt.com "[PDF] exe4j Manual - ej-technologies"))。

---

## 小結

1. **打包 JAR**：透過 `javac` + `jar cfm` 或 IDE 的 Runnable JAR 導出功能，產生含 Main-Class 的 `MyApp.jar`。
    
2. **exe4j 配置**：在 Wizard 中選擇「JAR in EXE」，指定主類別、類路徑、JRE、圖示等，並生成 .exe 分發檔。
    

按照上述流程，就能將現有 Java 專案先轉為執行 JAR，再透過 exe4j 打包成 Windows 原生可執行檔，方便在無 Java 環境的機器上直接使用。祝您成功！