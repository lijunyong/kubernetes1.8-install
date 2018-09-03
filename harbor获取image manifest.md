# harbor获取镜像Manifest
```
[root@node2 ~]# curl -ikL -X GET -u admin:Ljy,1987 http://172.20.0.119/service/token?account=admin\&service=harbor-registry\&scope=repository:k8s/nginx:pull
HTTP/1.1 200 OK
Server: nginx/1.11.13
Date: Mon, 03 Sep 2018 09:51:54 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 1190
Connection: keep-alive
Set-Cookie: beegosessionID=86b479a55d4c65966dcf9666553adc3e; Path=/; HttpOnly

{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IkhJRkQ6UE9WRDpPUDdHOjRUSU46RUVOVzpCNlVQOlFTUFU6R0JGUDpIV0NKOlFCRzQ6RFhEQzo3TTdSIn0.eyJpc3MiOiJoYXJib3ItdG9rZW4taXNzdWVyIiwic3ViIjoiYWRtaW4iLCJhdWQiOiJoYXJib3ItcmVnaXN0cnkiLCJleHAiOjE1MzU5NzAxMTQsIm5iZiI6MTUzNTk2ODMxNCwiaWF0IjoxNTM1OTY4MzE0LCJqdGkiOiJRbjc4RG1aZXJ3S1RpNXowIiwiYWNjZXNzIjpbeyJ0eXBlIjoicmVwb3NpdG9yeSIsIm5hbWUiOiJrOHMvbmdpbngiLCJhY3Rpb25zIjpbInB1c2giLCIqIiwicHVsbCJdfV19.FWcqrIB4y6y5mVUYSOPa1b-vJTlt-EokDeuRYJhujnRUe4uTw7sh583CHJNenFiD6V8SSLTJ5JyQPNx-0V-HRQ1EDq41rEEDRzciyuouXavjoFoJMBouj6IF5s2T4ZYJ81KRFbEiGZOmWduYsxF14ITUL8V_5SQNeAFdb-eWjpBukbRHT0FUqwPclWwEQ7OXAY0fas3L9B77DuIGTR6iL4c7MFDxPRnRUiUAZEfIk9w_mIomZmXUB3nAMQ6IRD7Ft_GZQFJiFRJBr3FCvPwJTDNaMJNfLhs6yKdJ92Q7fBoLzvURUOHaviHlJlxcLIW7_kQVZog7qbrKeeXDEuLeKIvQtxC5mjw9WZf9LtPbfIfq3jPWVmHmx6SNeiMzuVMHi-mzFATSZD8uZUmfapVtiK_NfkwkWVV5_yx7rlRA0CPITRhO2Y738gohSpAzrPzKxbnLmwbfJfj67iclxUyRkr_kMYoGxNfbjKVlNcIhAKpeTMfijpOlnCUtkjuNdEoGGq1KfYOEhdNZV0L9L7gsYIPpqwBItKQd7tq22xWdKF0ZEKq-acErHN-LX3gUTCO8pLvPPQsWz917TzRN67UUuMf2hjWJUeAmvmV-rL7gu_d_VVoKl14wEcTWSWwT0tqZf2UZuVnaceM4uZSU3CIV4fc6DePp20j1odrbRLDZu8c",
  "expires_in": 1800,
  "issued_at": "2018-09-03T09:51:54Z"
}[root@node2 ~]# curl -ikL -X GET -H "Content-Type: application/json" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IkhJRkQ6UE9WRDpPUDdHOjRUSU46RUVOVzpCNlVQOlFTUFU6RJGUDpIV0NKOlFCRzQ6RFhEQzo3TTdSIn0.eyJpc3MiOiJoYXJib3ItdG9rZW4taXNzdWVyIiwic3ViIjoiYWRtaW4iLCJhdWQiOiJoYXJib3ItcmVnaXN0cnkiLCJleHAiOjE1MzU5NzAxMTQsIm5iZiI6MTUzNTk2ODMxNCwiaWF0IjoxNTM1OTY4MzE0LCJqdGkiOiJRbjc4RG1aZXJ3S1RpNXowIiwiYWNjZXNzIjpbeyJ0eXBlIjoicmVwb3NpdG9yeSIsIm5hbWUiOiJrOHMvbmdpbngiLCJhY3Rpb25zIjpbInB1c2giLCIqIiwicHVsbCJdfV19.FWcqrIB4y6y5mVUYSOPa1b-vJTlt-EokDeuRYJhujnRUe4uTw7sh583CHJNenFiD6V8SSLTJ5JyQPNx-0V-HRQ1EDq41rEEDRzciyuouXavjoFoJMBouj6IF5s2T4ZYJ81KRFbEiGZOmWduYsxF14ITUL8V_5SQNeAFdb-eWjpBukbRHT0FUqwPclWwEQ7OXAY0fas3L9B77DuIGTR6iL4c7MFDxPRnRUiUAZEfIk9w_mIomZmXUB3nAMQ6IRD7Ft_GZQFJiFRJBr3FCvPwJTDNaMJNfLhs6yKdJ92Q7fBoLzvURUOHaviHlJlxcLIW7_kQVZog7qbrKeeXDEuLeKIvQtxC5mjw9WZf9LtPbfIfq3jPWVmHmx6SNeiMzuVMHi-mzFATSZD8uZUmfapVtiK_NfkwkWVV5_yx7rlRA0CPITRhO2Y738gohSpAzrPzKxbnLmwbfJfj67iclxUyRkr_kMYoGxNfbjKVlNcIhAKpeTMfijpOlnCUtkjuNdEoGGq1KfYOEhdNZV0L9L7gsYIPpqwBItKQd7tq22xWdKF0ZEKq-acErHN-LX3gUTCO8pLvPPQsWz917TzRN67UUuMf2hjWJUeAmvmV-rL7gu_d_VVoKl14wEcTWSWwT0tqZf2UZuVnaceM4uZSU3CIV4fc6DePp20j1odrbRLDZu8c" http://172.20.0.119/v2/k8s/nginx/tags/list
HTTP/1.1 200 OK
Server: nginx/1.11.13
Date: Mon, 03 Sep 2018 09:53:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 45
Connection: keep-alive
Docker-Distribution-Api-Version: registry/2.0
Set-Cookie: beegosessionID=ccbbdd430471b9b929075499bbca62e6; Path=/; HttpOnly

{"name":"k8s/nginx","tags":["latest","1.9"]}


[root@node2 ~]# curl -ikL -X GET -H "Content-Type: application/json" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IkhJRkQ6UE9WRDpPUDdHOjRUSU46RUVOVzpCNlVQOlFTUFU6R0JGUDpIV0NKOlFCRzQ6RFhEQzo3TTdSIn0.eyJpc3MiOiJoYXJib3ItdG9rZW4taXNzdWVyIiwic3ViIjoiYWRtaW4iLCJhdWQiOiJoYXJib3ItcmVnaXN0cnkiLCJleHAiOjE1MzU5NzAxMTQsIm5iZiI6MTUzNTk2ODMxNCwiaWF0IjoxNTM1OTY4MzE0LCJqdGkiOiJRbjc4RG1aZXJ3S1RpNXowIiwiYWNjZXNzIjpbeyJ0eXBlIjoicmVwb3NpdG9yeSIsIm5hbWUiOiJrOHMvbmdpbngiLCJhY3Rpb25zIjpbInB1c2giLCIqIiwicHVsbCJdfV19.FWcqrIB4y6y5mVUYSOPa1b-vJTlt-EokDeuRYJhujnRUe4uTw7sh583CHJNenFiD6V8SSLTJ5JyQPNx-0V-HRQ1EDq41rEEDRzciyuouXavjoFoJMBouj6IF5s2T4ZYJ81KRFbEiGZOmWduYsxF14ITUL8V_5SQNeAFdb-eWjpBukbRHT0FUqwPclWwEQ7OXAY0fas3L9B77DuIGTR6iL4c7MFDxPRnRUiUAZEfIk9w_mIomZmXUB3nAMQ6IRD7Ft_GZQFJiFRJBr3FCvPwJTDNaMJNfLhs6yKdJ92Q7fBoLzvURUOHaviHlJlxcLIW7_kQVZog7qbrKeeXDEuLeKIvQtxC5mjw9WZf9LtPbfIfq3jPWVmHmx6SNeiMzuVMHi-mzFATSZD8uZUmfapVtiK_NfkwkWVV5_yx7rlRA0CPITRhO2Y738gohSpAzrPzKxbnLmwbfJfj67iclxUyRkr_kMYoGxNfbjKVlNcIhAKpeTMfijpOlnCUtkjuNdEoGGq1KfYOEhdNZV0L9L7gsYIPpqwBItKQd7tq22xWdKF0ZEKq-acErHN-LX3gUTCO8pLvPPQsWz917TzRN67UUuMf2hjWJUeAmvmV-rL7gu_d_VVoKl14wEcTWSWwT0tqZf2UZuVnaceM4uZSU3CIV4fc6DePp20j1odrbRLDZu8c" http://172.20.0.119/v2/k8s/nginx/manifests/1.9
HTTP/1.1 200 OK
Server: nginx/1.11.13
Date: Mon, 03 Sep 2018 09:53:49 GMT
Content-Type: application/vnd.docker.distribution.manifest.v1+prettyjws
Content-Length: 6769
Connection: keep-alive
Docker-Content-Digest: sha256:ad6e97e5c2ad173f31428bca04d28930ae962ad4c283f63bb77c96db29c26357
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:ad6e97e5c2ad173f31428bca04d28930ae962ad4c283f63bb77c96db29c26357"
Set-Cookie: beegosessionID=a72d4c00c759516d26cb7ca0c7fd9286; Path=/; HttpOnly

{
   "schemaVersion": 1,
   "name": "k8s/nginx",
   "tag": "1.9",
   "architecture": "amd64",
   "fsLayers": [
      {
         "blobSum": "sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1"
      },
      {
         "blobSum": "sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1"
      },
      {
         "blobSum": "sha256:993553556507adfd4cf02c97e3885fc4bb54d8dc05bf2eee66d62cbc6e0fc1c8"
      },
      {
         "blobSum": "sha256:d77265f79c9eb251c74c76cc50eb33ae1abe40c7745871268f2ed879ee4eda5c"
      },
      {
         "blobSum": "sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1"
      },
      {
         "blobSum": "sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1"
      },
      {
         "blobSum": "sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1"
      },
      {
         "blobSum": "sha256:6e3caef028afd5180a08d0f24b6e4d7bbdbf79449408579c917f81f2f8a55db3"
      }
   ],
   "history": [
      {
         "v1Compatibility": "{\"architecture\":\"amd64\",\"author\":\"NGINX Docker Maintainers \\\"docker-maint@nginx.com\\\"\",\"config\":{\"Hostname\":\"b0cf605c7757\",\"Domainname\":\"\",\"User\":\"\",\"AttachStdin\":false,\"AttachStdout\":false,\"AttachStderr\":false,\"ExposedPorts\":{\"443/tcp\":{},\"80/tcp\":{}},\"Tty\":false,\"OpenStdin\":false,\"StdinOnce\":false,\"Env\":[\"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\",\"NGINX_VERSION=1.9.15-1~jessie\"],\"Cmd\":[\"nginx\",\"-g\",\"daemon off;\"],\"Image\":\"aa527287a51f0178662d50479697b53893b65fee9383af889ece937fd02c7c56\",\"Volumes\":null,\"WorkingDir\":\"\",\"Entrypoint\":null,\"OnBuild\":[],\"Labels\":{}},\"container\":\"b5b9e60402f2a7222d9c26ac26518d443e8dcd5cebda4e6d18abf20bffc4f11c\",\"container_config\":{\"Hostname\":\"b0cf605c7757\",\"Domainname\":\"\",\"User\":\"\",\"AttachStdin\":false,\"AttachStdout\":false,\"AttachStderr\":false,\"ExposedPorts\":{\"443/tcp\":{},\"80/tcp\":{}},\"Tty\":false,\"OpenStdin\":false,\"StdinOnce\":false,\"Env\":[\"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\",\"NGINX_VERSION=1.9.15-1~jessie\"],\"Cmd\":[\"/bin/sh\",\"-c\",\"#(nop) CMD [\\\"nginx\\\" \\\"-g\\\" \\\"daemon off;\\\"]\"],\"Image\":\"aa527287a51f0178662d50479697b53893b65fee9383af889ece937fd02c7c56\",\"Volumes\":null,\"WorkingDir\":\"\",\"Entrypoint\":null,\"OnBuild\":[],\"Labels\":{}},\"created\":\"2016-05-24T04:16:49.228757773Z\",\"docker_version\":\"1.9.1\",\"id\":\"9ca180bab5db0eb28cb4c2cf6303b19f2d3b7efd583de0bf9003f862ec53c860\",\"os\":\"linux\",\"parent\":\"77ec9e6a1b42c7af2eabb9c3cfd93af8a72e363635a29bfbdb6a7c7d1cbb9fd4\"}"
      },
      {
         "v1Compatibility": "{\"id\":\"77ec9e6a1b42c7af2eabb9c3cfd93af8a72e363635a29bfbdb6a7c7d1cbb9fd4\",\"parent\":\"f6c58441afbb8c38bb8cc70c46d3548603a3fea3973f934a0d54616cbe4b455d\",\"created\":\"2016-05-24T04:16:48.052193209Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) EXPOSE 443/tcp 80/tcp\"]},\"author\":\"NGINX Docker Maintainers \\\"docker-maint@nginx.com\\\"\"}"
      },
      {
         "v1Compatibility": "{\"id\":\"f6c58441afbb8c38bb8cc70c46d3548603a3fea3973f934a0d54616cbe4b455d\",\"parent\":\"e2f99431698ab03367e9f03acce855db1f446ff5de2d9b41d0f825fe5580282e\",\"created\":\"2016-05-24T04:16:46.665763882Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c ln -sf /dev/stdout /var/log/nginx/access.log \\t\\u0026\\u0026 ln -sf /dev/stderr /var/log/nginx/error.log\"]},\"author\":\"NGINX Docker Maintainers \\\"docker-maint@nginx.com\\\"\"}"
      },
      {
         "v1Compatibility": "{\"id\":\"e2f99431698ab03367e9f03acce855db1f446ff5de2d9b41d0f825fe5580282e\",\"parent\":\"1f4b824b4dc9b71d1c6b3088b18709527e942b50211e3ad36a26153496ffe8e0\",\"created\":\"2016-05-24T04:16:42.657465397Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62 \\t\\u0026\\u0026 echo \\\"deb http://nginx.org/packages/mainline/debian/ jessie nginx\\\" \\u003e\\u003e /etc/apt/sources.list \\t\\u0026\\u0026 apt-get update \\t\\u0026\\u0026 apt-get install --no-install-recommends --no-install-suggests -y \\t\\t\\t\\t\\t\\tca-certificates \\t\\t\\t\\t\\t\\tnginx=${NGINX_VERSION} \\t\\t\\t\\t\\t\\tnginx-module-xslt \\t\\t\\t\\t\\t\\tnginx-module-geoip \\t\\t\\t\\t\\t\\tnginx-module-image-filter \\t\\t\\t\\t\\t\\tnginx-module-perl \\t\\t\\t\\t\\t\\tnginx-module-njs \\t\\t\\t\\t\\t\\tgettext-base \\t\\u0026\\u0026 rm -rf /var/lib/apt/lists/*\"]},\"author\":\"NGINX Docker Maintainers \\\"docker-maint@nginx.com\\\"\"}"
      },
      {
         "v1Compatibility": "{\"id\":\"1f4b824b4dc9b71d1c6b3088b18709527e942b50211e3ad36a26153496ffe8e0\",\"parent\":\"5f264841fcf61e0efa850ea8cd49bf3de68c8738aa763e4c0ce3de88f787874a\",\"created\":\"2016-05-24T04:15:26.367439086Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) ENV NGINX_VERSION=1.9.15-1~jessie\"]},\"author\":\"NGINX Docker Maintainers \\\"docker-maint@nginx.com\\\"\"}"
      },
      {
         "v1Compatibility": "{\"id\":\"5f264841fcf61e0efa850ea8cd49bf3de68c8738aa763e4c0ce3de88f787874a\",\"parent\":\"8113625b0422d0d83c1987b91c0ba4577dcbada3dc3a0cbd32c337ef467c1b87\",\"created\":\"2016-05-24T04:15:25.236059965Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) MAINTAINER NGINX Docker Maintainers \\\"docker-maint@nginx.com\\\"\"]},\"author\":\"NGINX Docker Maintainers \\\"docker-maint@nginx.com\\\"\"}"
      },
      {
         "v1Compatibility": "{\"id\":\"8113625b0422d0d83c1987b91c0ba4577dcbada3dc3a0cbd32c337ef467c1b87\",\"parent\":\"b035fa1d799385f466a03d4d87692a6e83ce35996b41f7fab9cf00749e3bf465\",\"created\":\"2016-05-23T22:57:23.255025884Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) CMD [\\\"/bin/bash\\\"]\"]}}"
      },
      {
         "v1Compatibility": "{\"id\":\"b035fa1d799385f466a03d4d87692a6e83ce35996b41f7fab9cf00749e3bf465\",\"created\":\"2016-05-23T22:57:20.19311015Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) ADD file:5d8521419ad6cfb6956ed26ab70a44422d512f82462046ba8e68b7dcb8283f7e in /\"]}}"
      }
   ],
   "signatures": [
      {
         "header": {
            "jwk": {
               "crv": "P-256",
               "kid": "L6XL:QVO4:3YHZ:JPSG:I7AV:ITIX:52JM:YAEH:2PJM:KKIS:57WC:LFVS",
               "kty": "EC",
               "x": "_6e86EVbVcZw42_W3kPS_XjnqtuJI-_UtPlqJIqlJ4I",
               "y": "4xffJpKuoSlq7r4WAMyPOrEnvMwXKaJHj70PjZQpy6o"
            },
            "alg": "ES256"
         },
         "signature": "GyuwXn7xt7wlTFIKTb98L6QVEU--kyGJrsairrPN2GTp7nXGxv9UHOyXl29DyeX1Dmb1Wf6a_XimBHiWkYe6IQ",
         "protected": "eyJmb3JtYXRMZW5ndGgiOjYxMjIsImZvcm1hdFRhaWwiOiJDbjAiLCJ0aW1lIjoiMjAxOC0wOS0wM1QwOTo1Mzo0OVoifQ"
      }
   ]
}[root@node2 ~]# 

```
