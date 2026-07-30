FROM registry.cn-shenzhen.aliyuncs.com/lipaysam/dify-sandbox:0.2.15

# ffmpeg pulls in x11-common whose postinst needs id command; pre-install lsb-base to satisfy it
RUN apt-get update && \
    apt-get install -y --no-install-recommends lsb-base && \
    apt-get install -y --no-install-recommends ffmpeg && \
    rm -rf /var/lib/apt/lists/*
