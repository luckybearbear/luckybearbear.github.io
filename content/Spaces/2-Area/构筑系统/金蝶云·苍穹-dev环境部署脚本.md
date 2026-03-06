---
publish: true
aliases: ""
created: 2026-01-05
modified: 2026-01-05
cssclasses: ""
---


## 使用说明

```bash
python deploy_dev_small.py
```

## deploy_dev_small.py

```python

import os
import subprocess
import zipfile
from scp import SCPClient
import paramiko

def run_gradle_task(project_dir, task_name):
    """
    执行指定目录下的 Gradle 任务。
    :param project_dir: Gradle 项目目录
    :param task_name: 要执行的 Gradle 任务名称
    """
    print(f"正在执行 Gradle 任务 '{task_name}'…")
    gradlew_path = os.path.join(project_dir, "gradlew.bat")
    if not os.path.exists(gradlew_path):
        raise FileNotFoundError(f"未找到 Gradle Wrapper 文件: {gradlew_path}")

    try:
        result = subprocess.run([gradlew_path, task_name], cwd=project_dir, check=True)
        print(f"Gradle 任务 '{task_name}' 执行成功。")
    except subprocess.CalledProcessError as e:
        print(f"Gradle 任务 '{task_name}' 执行失败: {e}")
        raise

def compress_jars_to_zip(source_dir, output_zip):
    """
    将指定目录下的所有 .jar 文件压缩为 ZIP 文件。
    :param source_dir: 包含 .jar 文件的源目录
    :param output_zip: 输出的 ZIP 文件路径
    """
    print(f"正在将 {source_dir} 下的所有 .jar 文件压缩为 {output_zip}…")
    if not os.path.exists(source_dir):
        raise FileNotFoundError(f"源目录不存在: {source_dir}")

    jar_files = [f for f in os.listdir(source_dir) if f.endswith(".jar")]
    if not jar_files:
        raise FileNotFoundError(f"源目录中未找到任何 .jar 文件: {source_dir}")

    with zipfile.ZipFile(output_zip, 'w', zipfile.ZIP_DEFLATED) as zipf:
        for jar_file in jar_files:
            jar_path = os.path.join(source_dir, jar_file)
            zipf.write(jar_path, arcname=os.path.basename(jar_path))
            print(f"已添加文件: {jar_path}")

    print(f"压缩完成，输出文件: {output_zip}")


def upload_zip_via_scp(zip_file, remote_host, remote_user, remote_password, remote_path):
    """
    使用 SCP 将 ZIP 文件上传到远程服务器。
    :param zip_file: 本地 ZIP 文件路径
    :param remote_host: 远程服务器地址
    :param remote_user: 远程服务器用户名
    :param remote_password: 远程服务器密码
    :param remote_path: 远程服务器目标路径
    """
    print(f"正在将 {zip_file} 上传到 {remote_host}:{remote_path}…")
    if not os.path.exists(zip_file):
        raise FileNotFoundError(f"本地 ZIP 文件不存在: {zip_file}")

    try:
        # 创建 SSH 客户端
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        print(f"尝试连接到 {remote_host}…")
        ssh.connect(remote_host, username=remote_user, password=remote_password)

        # 检查并创建远程路径
        print(f"检查远程路径 {remote_path} 是否存在…")
        stdin, stdout, stderr = ssh.exec_command(f"mkdir -p {remote_path}")
        if stdout.channel.recv_exit_status() != 0:
            print(f"创建目录失败: {stderr.read().decode()}")

        # 使用 SCP 上传文件
        with SCPClient(ssh.get_transport()) as scp:
            remote_file = os.path.join(remote_path, os.path.basename(zip_file))
            print(f"上传文件到 {remote_host}:{remote_file}…")
            scp.put(zip_file, remote_path)
            print(f"文件已上传到 {remote_host}:{remote_file}")

        # 关闭连接
        ssh.close()

    except Exception as e:
        print(f"SCP 上传失败: {e}")
        raise
        
def restart_k8s_service_via_ssh(host, username, password, service_name, namespace='yongrui'):
    """
    通过SSH连接远程服务器重启K8s服务
    """
    # 创建SSH客户端
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    
    try:
        # 连接远程服务器
        ssh.connect(host, username=username, password=password)
        
        # 重启K8s服务的命令
        command = f"kubectl rollout restart deployment/{service_name} -n {namespace}"
        
        # 执行命令
        stdin, stdout, stderr = ssh.exec_command(command)
        
        # 获取执行结果
        output = stdout.read().decode('utf-8')
        error = stderr.read().decode('utf-8')
        
        if error:
            print(f"Error: {error}")
        else:
            print(f"Success: {output}")
            
    except Exception as e:
        print(f"Connection failed: {e}")
    finally:
        ssh.close()





if __name__ == "__main__":
    # 配置路径
    project_dir = r"D:\devloper\project\pxtc-yongrui"
    repository_dir = r"D:\project\kingdee\repository\mservice-cosmic\lib\cus"
    output_zip = r"D:\project\kingdee\repository\mservice-cosmic\lib\cus\project.zip"
    remote_host = "139.224.136.217"
    remote_user = "root"
    remote_password = "moldfun123!"
    remote_path = "/var/appstatic/appstore-yongrui/cosmic/cus"
    

    try:
        # 1. 执行 Gradle 任务 deployJar
        run_gradle_task(project_dir, "deployJar")

        # 2. 压缩 .jar 文件为 ZIP
        compress_jars_to_zip(repository_dir, output_zip)

        # 3. 使用 SCP 上传 ZIP 文件
        upload_zip_via_scp(output_zip, remote_host, remote_user, remote_password, remote_path)
        
        # 4. 重启K8s服务
        restart_k8s_service_via_ssh(remote_host, remote_user, remote_password, 'mservice-yongrui')
        restart_k8s_service_via_ssh(remote_host, remote_user, remote_password, 'web-yongrui')

    except Exception as e:
        print(f"发生错误: {e}")
```

## 元数据重建

[[Spaces/2-Area/构筑系统/金蝶云·苍穹-元数据重建]]
