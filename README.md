<p align="left">
    <a href="https://hub.docker.com/r/yershalom/aws-ecr-proxy" alt="Pulls">
        <img src="https://img.shields.io/docker/pulls/yershalom/aws-ecr-proxy" /></a>
</p>

# aws-ecr-http-proxy

A very simple nginx push/pull proxy that forwards requests to AWS ECR and caches the responses locally.

### Configuration:
The proxy is packaged in a docker container and can be configured with following environment variables:

| Environment Variable                | Description                                    | Status                            | Default    |
| :---------------------------------: | :--------------------------------------------: | :-------------------------------: | :--------: |
| `AWS_REGION`                        | AWS Region for AWS ECR                         | Required                          |            |
| `UPSTREAM`                          | URL for AWS ECR                                | Required                          |            |
| `RESOLVER`                          | DNS server to be used by proxy                 | Required                          |            |
| `PORT`                              | Port on which proxy listens                    | Required                          |            |
| `CACHE_MAX_SIZE`                    | Maximum size for cache volume                  | Optional                          |  `75g`     |
| `CACHE_KEY`                         | Cache key used for the content by nginx        | Optional                          |  `$uri`    |
| `ENABLE_SSL`                        | Used to enable SSL/TLS for proxy               | Optional                          | `false`    |
| `REGISTRY_HTTP_TLS_KEY`             | Path to TLS key in the container               | Required with TLS                 |            |
| `REGISTRY_HTTP_TLS_CERTIFICATE`     | Path to TLS cert in the container              | Required with TLS                 |            |

### Example:

```sh
docker run -d --name docker-registry-proxy --net=host \
  -v $(pwd)/cache:/cache \
  -v $(pwd)/roles/docker-registry-proxy/files/certificate.pem:/opt/ssl/certificate.pem \
  -v $(pwd)/roles/docker-registry-proxy/files/key.pem:/opt/ssl/key.pem \
  -e PORT=5000 \
  -e RESOLVER=8.8.8.8 \
  -e UPSTREAM=https://XXXXXXXXXX.dkr.ecr.eu-central-1.amazonaws.com \
  -e AWS_REGION=${AWS_DEFAULT_REGION} \
  -e CACHE_MAX_SIZE=100g \
  -e ENABLE_SSL=true \
  -e REGISTRY_HTTP_TLS_KEY=/opt/ssl/key.pem \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/opt/ssl/certificate.pem \
  yershalom/aws-ecr-proxy:latest
```

If you ran this command on "registry-proxy.example.com" you can now get your images using `docker pull registry-proxy.example.com:5000/repo/image`.


### Note on SSL/TLS
The proxy is using `HTTP` (plain text) as default protocol for now. So in order to avoid docker client complaining either:
 - (**Recommended**) Enable SSL/TLS using `ENABLE_SSL` configuration. For that you will have to mount your **valid** certificate/key in the container and pass the paths using  `REGISTRY_HTTP_TLS_*` variables.
 - Mark the registry host as insecure in your client [deamon config](https://docs.docker.com/registry/insecure/).
