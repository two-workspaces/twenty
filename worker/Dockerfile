FROM twentycrm/twenty:v2.56.1

# BullMQ worker: emails, imports, workflow runs. Migrations and cron
# registration stay on the server (upstream compose does the same).
ENV DISABLE_DB_MIGRATIONS=true \
    DISABLE_CRON_JOBS_REGISTRATION=true \
    STORAGE_TYPE=s3 \
    STORAGE_S3_REGION=auto

# Twenty reads the schema version once at boot. A worker that starts while the
# server is still upgrading keeps querying dropped columns until restarted, so
# wait for the server's upgrade to finish first.
#
# Under 2 GB, Node's default heap is half the limit. At 1 GB (Railway's Trial plan)
# that is about 480 MB, and the jobs that follow a new workspace run the worker out
# of heap, so it restarts while Railway still shows SUCCESS. Give it 70% of the limit.
CMD ["sh", "-c", "limit=$(cat /sys/fs/cgroup/memory.max 2>/dev/null); case \"$limit\" in ''|max) ;; *) if [ \"$limit\" -lt 2147483648 ]; then heap=$((limit / 1048576 * 7 / 10)); export NODE_OPTIONS=\"--max-old-space-size=$heap $NODE_OPTIONS\"; echo \"worker: heap set to $heap MB\"; fi ;; esac; until yarn command:prod upgrade:status 2>/dev/null | grep -q 'Instance: Up to date'; do echo 'worker: waiting for the server to finish upgrading the database'; sleep 10; done; exec yarn worker:prod"]
