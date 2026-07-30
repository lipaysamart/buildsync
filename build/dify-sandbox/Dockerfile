FROM registry.cn-shenzhen.aliyuncs.com/lipaysam/dify-sandbox:0.2.15

# ffmpeg pulls in x11-common whose postinst needs update-rc.d from init-system-helpers,
# which isn't available in the minimal base image; install it explicitly
RUN apt-get update && \
    apt-get install -y --no-install-recommends init-system-helpers ffmpeg && \
    rm -rf /var/lib/apt/lists/*
