
## Secure & Hardening images


0) If app permits use multi-stage build
1) Use specific package version of the image
2) Don't run as ROOT user
3) Make filesystem read only
4) Remove shell access



```Dockerfile
# Stage 1: Build the application
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

# Stage 2: Create a minimal image with the application
FROM alpine:3.12.1
RUN chmod a-r / # Reduce permissions
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
COPY --from=0 /app /home/appuser/
RUN rm -rf /bin/*
USER appuser
CMD ["/home/appuser/app"]

```