FROM registry.centos.org/centos/centos:7

# Install dependencies for mattermost-integration-github
RUN yum -y update && \
    yum -y install epel-release && \
    yum -y install nodejs && \
    yum -y install zlib.i686 libstdc++.i686 make git android-tools.x86_64 java-1.8.0-openjdk-devel java-1.8.0-openjdk-headless wget unzip

RUN cd /opt && \
    wget --output-document=android-sdk.zip --quiet https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip && \
    unzip android-sdk.zip && \
    rm -f android-sdk.zip && \
    mkdir android-sdk-linux && \
    mv tools/ android-sdk-linux/ && \
    chown -R root:root android-sdk-linux

RUN wget https://dl.yarnpkg.com/rpm/yarn.repo -O /etc/yum.repos.d/yarn.repo &&\
    yum -y install yarn

ENV ANDROID_HOME /opt/android-sdk-linux
ENV GRADLE_HOME /opt/gradle
ENV PATH ${PATH}:${ANDROID_HOME}/tools:${ANDROID_HOME}/tools/bin
ENV JAVA_HOME=/usr/lib/jvm/jre-openjdk/
# Install Android Platforms
RUN export JAVA_HOME=/usr/lib/jvm/jre-openjdk/ && \
echo y | sdkmanager "platform-tools" && \
echo " Installing APIS" && \
echo y | sdkmanager "add-ons;addon-google_apis-google-23" && \
echo y | sdkmanager "add-ons;addon-google_apis-google-24" && \
echo y | sdkmanager "add-ons;addon-google_apis-google-22" && \
echo y | sdkmanager "add-ons;addon-google_apis-google-21" && \
echo y | sdkmanager "platforms;android-25" && \
echo y | sdkmanager "platforms;android-23" && \
echo y | sdkmanager "build-tools;26.0.1" &&\
echo y | sdkmanager "build-tools;26.0.0" &&\
echo y | sdkmanager "build-tools;25.0.2" && \
echo y | sdkmanager "build-tools;25.0.1" && \
echo y | sdkmanager "build-tools;25.0.0" && \
echo y | sdkmanager "build-tools;24.0.0" && \
echo y | sdkmanager "build-tools;23.0.1" && \
echo y | sdkmanager "extras;android;m2repository" && \
echo y | sdkmanager "extras;google;m2repository" && \
echo y | sdkmanager "extras;google;google_play_services" && \
mkdir -p $ANDROID_HOME/licenses/ && \
echo "8933bad161af4178b1185d1a37fbf41ea5269c55" > $ANDROID_HOME/licenses/android-sdk-license && \
echo "84831b9409646a918e30573bab4c9c91346d8abd" > $ANDROID_HOME/licenses/android-sdk-preview-license


RUN cd /opt/ && \
version="$(curl --silent "https://api.github.com/repos/mattermost/mattermost-mobile/releases/latest" | grep -Po '"tag_name": "\K.*?(?=")')" &&\
wget https://github.com/mattermost/mattermost-mobile/archive/${version}.tar.gz &&\
tar xvf ${version}.tar.gz  && \
cd /opt/mattermost-mobile-${version:1} && \
yarn --ignore-engines install && \
make && cd android/keystores &&\   
keytool -genkey -noprompt -v -keystore  my-release-key.keystore \
-dname "CN=gustavo paez, OU=nimetrix, O=nimetrix, L=monterrey, ST=nuevo leon, C=mx" \
-keypass Wsp123 -storepass Wsp123  -alias alias_name -keyalg RSA -keysize 2048 -validity 10000 && \
cd / && cd  opt/android-sdk-linux/tools/bin &&\
yes | ./sdkmanager --licenses &&\
cd / && cd opt/mattermost-mobile-${version:1}/android/  && echo y | yum install vim-enhanced.x86_64 && \
vim +"$ | put! ='MATTERMOST_RELEASE_STORE_FILE=../keystores/my-release-key.keystore' | x" gradle.properties &&\
vim +"$ | put! = 'MATTERMOST_RELEASE_PASSWORD = Wsp123' | x" gradle.properties && \
vim +"$ | put! = 'MATTERMOST_RELEASE_KEY_ALIAS = alias_name' | x" gradle.properties && \
vim +"$ | put! = 'MATTERMOST_MARELEASE_KEY_PASSWORD = Wsp123' | x" gradle.properties &&\  
cd / && cd opt/mattermost-mobile-${version:1}/android && ./gradlew assembleRelease 
