# bank-ledger-webhook

I designed the system as an asynchronous event-driven pipeline.

The webhook API does not process the full transaction payload directly. It validates the request, generates a deterministic batch hash for idempotency, stores the batch metadata, pushes a job to a Redis/BullMQ queue, and returns 202 Accepted.

The worker then processes the batch in the background. It uses unique indexes to prevent duplicate transactions from being inserted. For ledger accuracy, transactions are sorted by date and transaction id, then the running balance is recalculated in deterministic order.

To avoid concurrency issues, I use account-level locking so two workers cannot update the same account ledger at the same time.

For unreliable third-party bank APIs, I would use rate limiting, retry with exponential backoff, and DLQ for failed messages.

In production, I would replace Redis/BullMQ with AWS SQS or Kafka, store huge payloads in S3, and use CloudWatch/X-Ray/OpenTelemetry for observability.
