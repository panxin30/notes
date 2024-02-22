https://www.jenkins.io/doc/book/installing/linux/

因为其他软件安装了java8，所以这个jenkins选择一个支持java8的版本
### Long Term Support release[](https://www.jenkins.io/doc/book/installing/linux/#long-term-support-release)

A[LTS (Long-Term Support) release](https://www.jenkins.io/download/lts/)is chosen every 12 weeks from the stream of regular releases as the stable release for that time period. It can be installed from the[`debian-stable`apt repository](https://pkg.jenkins.io/debian-stable/).

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt install jenkins=2.346.1
```