FROM twentycrm/twenty:v2.56.1

# API + web app. Runs DB migrations and registers cron jobs on boot
# (the worker has both disabled). Files go to the bundled Railway bucket —
# Twenty's S3 driver uses headBucket on the specific bucket, which works
# with Railway's single-bucket-scoped credentials (unlike minio-based apps).
ENV NODE_PORT=3000 \
    STORAGE_TYPE=s3 \
    STORAGE_S3_REGION=auto

# Under 2 GB, Node's default heap is half the limit. At 1 GB (Railway's Trial plan)
# that is about 480 MB, and creating the first workspace can run the server out of
# heap, so sign-up fails with 502s. Give it 70% of the limit, migrations included.
ENTRYPOINT ["sh", "-c", "limit=$(cat /sys/fs/cgroup/memory.max 2>/dev/null); case \"$limit\" in ''|max) ;; *) if [ \"$limit\" -lt 2147483648 ]; then heap=$((limit / 1048576 * 7 / 10)); export NODE_OPTIONS=\"--max-old-space-size=$heap $NODE_OPTIONS\"; echo \"server: heap set to $heap MB\"; fi ;; esac; exec /app/entrypoint.sh \"$@\"", "--"]
CMD ["node", "dist/main"]
