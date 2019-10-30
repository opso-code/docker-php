ARG NGINX_VERSION=stable
FROM nginx:${NGINX_VERSION}-alpine
LABEL maintainer="opso <945206413@qq.com>"

ARG TIMEZONE=Asia/Shanghai
RUN cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone

ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories \
;fi

RUN apk update \
    && apk upgrade \
    && apk add --no-cache openssl \
    && apk add --no-cache bash

RUN chown nginx:nginx /var/log/nginx

CMD ["nginx", "-g", "daemon off;"]