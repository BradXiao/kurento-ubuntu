# Kurento Media Server (KMS) for Ubuntu 22.04 LTS

This repo contains KMS source code from [kurentu](https://github.com/Kurento/kurento) with minor modifications and other dependencies forked on Dec. 19th, 2023. The pre-build binaries for Ubuntu 22.04 (amd64) are also provided.


Most of the modifications involve replacing 'gstreamer-1.0' with the older 'gstreamer-1.5' because the official KMS 7.0.2 doesn't work on Ubuntu 22.04. This is the temporary soultion and for test only. The modifications are listed in the following table.


|Directory/File|Source|Modified|
|--|--|--|
|gstreamer|[kurento](https://github.com/Kurento/gstreamer)|No|
|openssl-1.0.2u.tar.gz|[openssl](https://www.openssl.org/source/old/1.0.2/openssl-1.0.2u.tar.gz)|No|
|libnice|[kurento](https://github.com/Kurento/libnice)|[Yes](https://github.com/BradXiao/kurento-ubuntu/commit/9cb5e5ab7a3c500f936b1ed1904f6fc8c2ca90c1)|
|libsrtp|[kurento](https://github.com/Kurento/libsrtp)|No|
|openh264|[cisco](https://github.com/cisco/openh264)|No|
|gst-plugins-base|[kurento](https://github.com/Kurento/gst-plugins-base)|No|
|gst-plugins-good|[kurento](https://github.com/Kurento/gst-plugins-good)|[Yes](https://github.com/BradXiao/kurento-ubuntu/commit/e19d11e75ea4df9628546f04e7bc639fd3dd8ddf)|
|gst-plugins-bad|[kurento](https://github.com/Kurento/gst-plugins-bad)|No|
|kurento server|[kurento](https://github.com/Kurento/kurento)|[Yes](https://github.com/BradXiao/kurento-ubuntu/commit/0cf29111c75f455c02b21366a0c7634117443161)|


## Pre-build binaries (for normal users)

### Fast installation
The script will install necessary packages, extract KMS binaries and create a KMS systemd service.

```bash
./install.sh
```

### Package dependencies (for running only)
```
libboost-filesystem1.74.0
libboost-thread1.74.0
libboost-program-options1.74.0
libboost-log1.74.0
libjsoncpp25
libsigc++-2.0-0v5
libglibmm-2.4-1v5
libvpx7
libsoup2.4-1
libopencv-imgcodecs4.5d
libopencv-objdetect4.5d
opencv-data
libopus0
libusrsctp2
libgupnp-igd-1.0-4
```

## Usage

### KMS Service

- `systemctl start kms.service`
- `systemctl enable kms.service`

### Configurations

- ` /opt/kms/etc/kurento/kurento.conf.json`
- ` /opt/kms/etc/kurento/modules/kurento/*.ini`

[See kurento documentation for configurations](https://doc-kurento.readthedocs.io/en/latest/user/configuration.html)



### Log files
You can modify the file `/etc/systemd/system/kms.service`.

```ini
...
Environment="GST_DEBUG=*:3"
Environment="KURENTO_LOG_FILE_SIZE=20"
Environment="KURENTO_NUMBER_LOG_FILES=10"
Environment="KURENTO_LOGS_PATH=/var/log/kurento-media-server/"
...
```
[See kurento documentation for logging](https://doc-kurento.readthedocs.io/en/latest/features/logging.html)


### Other check list

- Firewall  
	The default port for KMS is 8888 which can be modified in `kurento.conf.json`. And also check RTP streaming settings in `BaseRtpEndpoint.conf.ini`.

	```bash
	# something like this
	ufw allow 8888/tcp
	```
- STUN/TURN server  
    The server parameters can be modified in the file `WebRtcEndpoint.conf.ini`.  
	[See kurento documentation for TURN/STUN](https://doc-kurento.readthedocs.io/en/latest/user/faq.html#faq-stun-needed)



## Build or development (for developers)

This is for devleopers who would like to build their own module.  

![](KMS-dependency.png)


### Package dependencies
```
maven
build-essential
ca-certificates
cmake
git
gnupg
pkg-config
openjdk-11-jdk-headless
libboost-all-dev
libsigc++-2.0-dev
libglibmm-2.4-dev
libvpx-dev
libsoup2.4-dev
gtk-doc-tools
libbison-dev
flex
autopoint
libjsoncpp-dev
libwebsocketpp-dev
libopencv-dev
libopus-dev
libusrsctp-dev
nasm
```

### Packages/components build order

> [!IMPORTANT]
> Not that the order matters.

1. `gstreamer`
2. `openssl`
3. `libnice`
4. `libsrtp`
5. `openh264`
6. `gst-plugins-base`
7. `gst-plugins-good`
8. `gst-plugins-bad`
9. `openwebrtc-gst-plugins`
10. `module-creator`
11. `cmake-utils`
12. `jsonrpc`
13. `module-core`
14. `module-elements`
15. `module-filters`
16. `media-server`

### Java client

Before building KMS Java client, you will need to configure GitHub Packages in `~/.m2/settings.xml`.   
Replace the USERNAME and ACCESS TOKEN with yours. 
[See more info](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry).
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">

  <activeProfiles>
    <activeProfile>default</activeProfile>
  </activeProfiles>
  <profiles>
    <profile>
      <id>default</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>https://repo1.maven.org/maven2</url>
        </repository>
        <repository>
          <id>default</id>
          <url>https://maven.pkg.github.com/Kurento/kurento</url>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
  <servers>
    <server>
      <id>default</id>
      <username>USERNAME</username>
      <password>ACCESS TOKEN</password>
    </server>
  </servers>
</settings>
```

```bash
cd kurento/clients/java
mvn install
```

### Run tutorials

Only some of the tutorials are included in this repo. [See more.](https://github.com/Kurento/kurento/tree/main/tutorials/java)  
The following examples are tested.
- hello-world
- hello-world-recording
- magic-mirror
```bash
cd kurento/tutorials/java/hell-world

mvn -U clean spring-boot:run \
	-Dspring-boot.run.jvmArguments="-Dkms.url=ws://{YOUR KMS IP}:8888/kurento"
```

Browse https://{YOUR AP IP}:8443/.




## Disclaimer

THIS SOFTWARE IS PROVIDED "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

