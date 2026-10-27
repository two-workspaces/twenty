FROM twentycrm/twenty:v2.51.0

# BullMQ worker: emails, imports, workflow runs. Migrations and cron
# registration stay on the server (upstream compose does the same).
ENV DISABLE_DB_MIGRATIONS=true \
    DISABLE_CRON_JOBS_REGISTRATION=true \
    STORAGE_TYPE=s3 \
    STORAGE_S3_REGION=auto

# Twenty reads the schema version once at boot. A worker that starts while the
# server is still upgrading keeps querying dropped columns until restarted, so
# wait for the server's upgrade to finish first.
CMD ["sh", "-c", "until yarn command:prod upgrade:status 2>/dev/null | grep -q 'Instance: Up to date'; do echo 'worker: waiting for the server to finish upgrading the database'; sleep 10; done; exec yarn worker:prod"]
