FROM twentycrm/twenty:v2.51.0

# API + web app. Runs DB migrations and registers cron jobs on boot
# (the worker has both disabled). Files go to the bundled Railway bucket —
# Twenty's S3 driver uses headBucket on the specific bucket, which works
# with Railway's single-bucket-scoped credentials (unlike minio-based apps).
ENV NODE_PORT=3000 \
    STORAGE_TYPE=s3 \
    STORAGE_S3_REGION=auto
