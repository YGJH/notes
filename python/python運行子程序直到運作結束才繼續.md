您可以通过以下几种方法来确保 `subprocess.run` 完成后再重试：

## 方法1：添加等待时间和检查机制

````python
// ...existing code...
if copy:
    import platform
    import shutil
    if platform.system() == 'Linux':
        # cmd = 'cp summary.md ' + path + 'documents/obsidian/summary.md'  
        try:
            shutil.copy('summary.md', '/home/charles/pCloudDrive/documents/obsidian/summary.md')
        except Exception as e:
            print("pCloud not mounted, starting pCloud...")
            # 启动 pCloud 并等待
            process = subprocess.Popen('/home/charles/Downloads/pcloud', shell=True)
            
            # 等待 pCloud 挂载完成
            max_wait_time = 60  # 最大等待60秒
            wait_interval = 2   # 每2秒检查一次
            
            for i in range(0, max_wait_time, wait_interval):
                time.sleep(wait_interval)
                # 检查目录是否存在（说明已挂载）
                if os.path.exists('/home/charles/pCloudDrive/documents/obsidian/'):
                    print(f"pCloud mounted successfully after {i+wait_interval} seconds")
                    break
                print(f"Waiting for pCloud to mount... ({i+wait_interval}s)")
            else:
                print("Timeout waiting for pCloud to mount")
                return
            
            # 重试复制
            try:
                shutil.copy('summary.md', '/home/charles/pCloudDrive/documents/obsidian/summary.md')
                print("File copied successfully to pCloud")
            except Exception as e2:
                print(f"Failed to copy after pCloud startup: {e2}")
// ...existing code...
````

## 方法2：使用重试机制

````python
// ...existing code...
if copy:
    import platform
    import shutil
    
    def copy_with_retry(source, dest, max_retries=3):
        """尝试复制文件，如果失败则启动 pCloud 并重试"""
        for attempt in range(max_retries):
            try:
                shutil.copy(source, dest)
                print(f"File copied successfully on attempt {attempt + 1}")
                return True
            except Exception as e:
                print(f"Copy attempt {attempt + 1} failed: {e}")
                
                if attempt < max_retries - 1:  # 不是最后一次尝试
                    print("Starting pCloud and waiting...")
                    # 启动 pCloud
                    subprocess.run('/home/charles/Downloads/pcloud', shell=True)
                    
                    # 等待挂载
                    time.sleep(10)  # 等待10秒让 pCloud 启动
                    
                    # 检查是否挂载成功
                    if os.path.exists(os.path.dirname(dest)):
                        print("pCloud appears to be mounted, retrying...")
                        continue
                    else:
                        print("pCloud directory still not available, waiting longer...")
                        time.sleep(20)  # 再等待20秒
                else:
                    print("All retry attempts failed")
                    return False
        return False
    
    if platform.system() == 'Linux':
        success = copy_with_retry('summary.md', '/home/charles/pCloudDrive/documents/obsidian/summary.md')
        if not success:
            print("Failed to copy file to pCloud after all attempts")
            
    elif platform.system() == 'Windows':
        shutil.copy('summary.md', 'P:\\documents\\obsidian\\summary.md')
    else:
        print("We don't have this man here")
// ...existing code...
````

## 方法3：使用 subprocess.Popen 进行更好的控制

````python
// ...existing code...
if copy:
    import platform
    import shutil
    
    def wait_for_pcloud_mount():
        """等待 pCloud 挂载完成"""
        mount_path = '/home/charles/pCloudDrive'
        target_dir = '/home/charles/pCloudDrive/documents/obsidian/'
        
        print("Starting pCloud...")
        # 使用 Popen 启动 pCloud，但不等待它完成
        process = subprocess.Popen(['/home/charles/Downloads/pcloud'])
        
        # 等待挂载点出现
        max_attempts = 30  # 最多等待60秒 (30 * 2秒)
        for attempt in range(max_attempts):
            if os.path.ismount(mount_path) or os.path.exists(target_dir):
                print(f"pCloud mounted successfully after {attempt * 2} seconds")
                return True
            
            print(f"Waiting for pCloud mount... ({attempt * 2}s)")
            time.sleep(2)
        
        print("Timeout: pCloud failed to mount within expected time")
        return False
    
    if platform.system() == 'Linux':
        try:
            shutil.copy('summary.md', '/home/charles/pCloudDrive/documents/obsidian/summary.md')
            print("File copied successfully")
        except Exception as e:
            print(f"Initial copy failed: {e}")
            
            if wait_for_pcloud_mount():
                try:
                    shutil.copy('summary.md', '/home/charles/pCloudDrive/documents/obsidian/summary.md')
                    print("File copied successfully after pCloud mount")
                except Exception as e2:
                    print(f"Copy failed even after mounting: {e2}")
            else:
                print("Could not mount pCloud, copy operation failed")
                
    elif platform.system() == 'Windows':
        shutil.copy('summary.md', 'P:\\documents\\obsidian\\summary.md')
    else:
        print("We don't have this man here")
// ...existing code...
````

推荐使用**方法3**，因为它：
1. 使用 `subprocess.Popen` 启动 pCloud 但不阻塞
2. 通过检查挂载点或目录存在来确认 pCloud 是否就绪
3. 有明确的超时机制
4. 提供详细的状态反馈

这样可以确保 pCloud 完全启动并挂载后再进行文件复制操作。