FROM curlimages/curl:latest
MAINTAINER eitanas1 <eitanas1@gmail.com>

USER curl_user
WORKDIR /home/curl_user/app

EXPOSE 8080

ENV LOG_LEVEL=INFO \
    USERNAME=username \
    USER_PASSWORD=user_password

COPY --chown=curl_user:curl_group web-calculator .

CMD ["/home/curl_user/app/web-calculator"]
