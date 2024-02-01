FROM python:3.11-slim AS base

RUN sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list.d/debian.sources

RUN sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list.d/debian.sources

ENV PATH "/opt/venv/bin:$PATH"
ENV PYTHONUNBUFFERED True
ENV PYTHONDONTWRITEBYTECODE True
ENV PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION 'python'
ENV PYTHONPATH "/data:/data/vendor:/data/service"

FROM base AS builder
RUN python -m venv /opt/venv
ADD . /data
RUN apt-get update && apt-get install --yes --no-install-recommends libprotobuf-dev protobuf-compiler
RUN cd /data && mv ./.env.example ./.env && chmod +x ./bp && ./bp && pip install --no-cache-dir --upgrade pip -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

FROM base

COPY --from=builder /opt/venv /opt/venv
COPY --from=builder /data /data
RUN apt-get update && apt-get install --yes --no-install-recommends libjemalloc2
ENV LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2 MALLOC_CONF='background_thread:true,dirty_decay_ms:0,muzzy_decay_ms:0'

CMD ["uvicorn","main:app","--host","0.0.0.0","--log-config","logger_config.yaml","--no-access-log"]
