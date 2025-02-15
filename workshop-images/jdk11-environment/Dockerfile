#syntax=docker/dockerfile:1.3-labs

ARG IMAGE_REPOSITORY=localhost:5001
ARG BASE_IMAGE_NAME=educates-base-environment
ARG PACKAGE_VERSION=latest

FROM ${IMAGE_REPOSITORY}/${BASE_IMAGE_NAME}:${PACKAGE_VERSION} AS scratch-image

ARG TARGETARCH

RUN mkdir -p /opt/{java,gradle,maven}

RUN <<EOF
    set -eo pipefail
    ARCHNAME_amd64=x64
    ARCHNAME_arm64=aarch64
    ARCHNAME=ARCHNAME_${TARGETARCH}
    CHECKSUM_amd64="5f19fb28aea3e28fcc402b73ce72f62b602992d48769502effe81c52ca39a581"
    CHECKSUM_arm64="0c7763a19b4af4ef5fbae831781b5184e988d6f131d264482399eeaf51b6e254"
    CHECKSUM=CHECKSUM_${TARGETARCH}
    curl --fail -sL -o /tmp/jdk11.tar.gz https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.19%2B7/OpenJDK11U-jdk_${!ARCHNAME}_linux_hotspot_11.0.19_7.tar.gz
    echo "${!CHECKSUM} /tmp/jdk11.tar.gz" | sha256sum --check --status
    tar -C /opt/java --strip-components 1 -zxf /tmp/jdk11.tar.gz
    rm /tmp/jdk11.tar.gz
EOF

RUN curl --fail -sL -o /tmp/maven.tar.gz https://archive.apache.org/dist/maven/maven-3/3.9.2/binaries/apache-maven-3.9.2-bin.tar.gz && \
    echo "900bdeeeae550d2d2b3920fe0e00e41b0069f32c019d566465015bdd1b3866395cbe016e22d95d25d51d3a5e614af2c83ec9b282d73309f644859bbad08b63db /tmp/maven.tar.gz" | sha512sum --check --status && \
    tar -C /opt/maven --strip-components 1 -zxf /tmp/maven.tar.gz && \
    rm /tmp/maven.tar.gz

RUN curl --fail -sL -o /tmp/gradle.zip https://services.gradle.org/distributions/gradle-7.6.1-bin.zip && \
    echo "6147605a23b4eff6c334927a86ff3508cb5d6722cd624c97ded4c2e8640f1f87 /tmp/gradle.zip" | sha256sum --check --status && \
    unzip -d /opt/gradle /tmp/gradle.zip && \
    mv /opt/gradle/gradle-7.6.1/* /opt/gradle/ && \
    rm -rf /opt/gradle/gradle-7.6.1 && \
    rm /tmp/gradle.zip

ENV PATH=/opt/java/bin:/opt/gradle/bin:/opt/maven/bin:$PATH \
    JAVA_HOME=/opt/java \
    M2_HOME=/opt/maven

RUN mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app \
        -DarchetypeArtifactId=maven-archetype-quickstart \
        -DarchetypeVersion=1.4 -DinteractiveMode=false && \
    cd my-app && \
    mvn wrapper:wrapper

RUN gradle init && \
    gradle wrapper --gradle-version=7.4.2 --distribution-type=bin && \
    ./gradlew build

FROM ${IMAGE_REPOSITORY}/${BASE_IMAGE_NAME}:${PACKAGE_VERSION}

COPY --from=scratch-image --chown=1001:0 /opt/java /opt/java
COPY --from=scratch-image --chown=1001:0 /opt/gradle /opt/gradle
COPY --from=scratch-image --chown=1001:0 /opt/maven /opt/maven

COPY --from=scratch-image --chown=1001:0 /home/eduk8s/.m2 /home/eduk8s/.m2
COPY --from=scratch-image --chown=1001:0 /home/eduk8s/.gradle /home/eduk8s/.gradle

COPY --chown=1001:0 opt/. /opt/

ENV PATH=/opt/java/bin:/opt/gradle/bin:/opt/maven/bin:$PATH \
    JAVA_HOME=/opt/java \
    M2_HOME=/opt/maven
