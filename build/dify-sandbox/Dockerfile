FROM registry.cn-shenzhen.aliyuncs.com/lipaysam/dify-sandbox:0.2.15

# ffmpeg pulls in x11-common whose postinst needs update-rc.d (init-system-helpers)
# which isn't available in the minimal base image; use --force-confold to skip postinst
RUN apt-get update && \
    apt-get install -y --no-install-recommends -o Dpkg::Options::="--force-confold" ffmpeg && \
    rm -rf /var/lib/apt/lists/*
