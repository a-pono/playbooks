# Parquet Compression: Two Config Changes, Up to 50% Less Storage

Half your storage bill might be a config problem, not a data problem.

Most Spark jobs write Parquet with Snappy compression. Vertica does the same. It's been the default forever, and almost nobody questions it.

Switching to ZSTD is a one-line config change. No code rewrite, no schema changes, nothing breaks. That alone saves up to ~40% storage.

Then sort your data before writing. Pick columns based on your most common query filters - date, region, customer ID, whatever gets filtered the most. Sorting improves compression because similar values end up next to each other, and it tightens row group min/max statistics so the query engine can skip entire row groups through predicate pushdown.

That's up to another 20% on top.

Even ClickHouse, which already defaults to ZSTD, benefits significantly from sorting - both for better compression and predicate pushdown. The codec is only half the equation.

Sorting does add time to writes. If your workload is mostly writes and rarely reads, measure first. But most data pipelines write once and read many times, and that's where the tradeoff pays for itself quickly.

Two config-level changes, up to 50% less storage. What most people don't measure is the read side - Parquet almost always sits on network storage. S3, HDFS, NAS. Less data over the wire means faster reads. Up to 30% in my experience, because the time saved on network transfer more than covers the slower ZSTD decompression.

Compression codec + sort order, an afternoon of work at most.

If you're running Parquet at any real volume and haven't touched these settings, check your defaults.

## Examples

### Spark

```python
# Option 1: Set globally for all Parquet writes in this session
spark.conf.set("spark.sql.parquet.compression.codec", "zstd")

# Option 2: Set per-write
df.orderBy("date", "region", "customer_id") \
  .write \
  .option("compression", "zstd") \
  .mode("overwrite") \
  .parquet("s3://bucket/path/")

# Sort before writing - pick columns based on your most common query filters
df.orderBy("date", "region", "customer_id") \
  .write \
  .mode("overwrite") \
  .parquet("s3://bucket/path/")
```

### ClickHouse

```sql
-- ClickHouse already defaults to ZSTD for Parquet output,
-- but you still benefit from sorting

-- Sort on export
SELECT *
FROM my_table
ORDER BY date, region, customer_id
INTO OUTFILE 'data.parquet'
FORMAT Parquet

-- If you're on an older version that still defaults to LZ4/Snappy:
SET output_format_parquet_compression_method = 'zstd';
```

### Vertica

```sql
-- Switch from Snappy (default) to ZSTD
EXPORT TO PARQUET(
    directory = '/data/output',
    compression = 'ZSTD'
)
AS SELECT *
FROM my_table
ORDER BY date, region, customer_id;
```

## Why It Works

1. **ZSTD vs Snappy**: ZSTD achieves up to ~40% better compression at the cost of slower compression/decompression. On network storage (S3, HDFS, NAS), the reduced transfer time more than compensates.

2. **Sorting**: Columnar formats like Parquet compress better when similar values are adjacent. Sorting also tightens row group min/max statistics, enabling predicate pushdown - the query engine skips entire row groups that don't match your filters.
