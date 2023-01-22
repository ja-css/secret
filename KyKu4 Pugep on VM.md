## 1. Create VM
Create VM with certificate-protected ssh

## 2. Install java 17
`sudo apt update`
`sudo apt install openjdk-17-jre-headless`

## 3. Build jar
`gradle shadowJar -Dorg.gradle.java.home=/Users/john/Library/Java/JavaVirtualMachines/temurin-17.0.5/Contents/Home`

## 4. Copy jar
With sftp
`sftp -o "SmartcardDevice=/usr/local/lib/libeToken.dylib" ubuntu@ubuntu-server.aws.com`

## 5. Run jar with nohup
`nohup java -jar KyKu4-Pugep-all.jar -a > out.out &`

## 6. See the logs accumulate
Folder name `YYYY-MM-DD`

## 7. Get downloads
`sudo apt install zip`
`zip -r 2022-11-27.zip 2022-11-27`
Download with sftp