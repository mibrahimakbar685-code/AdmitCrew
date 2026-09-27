# Production backend: n8n + PostgreSQL

This folder adds the backend architecture required by the PRD. It is separate from GitHub Pages because a static site cannot safely store API keys or process WhatsApp webhooks.

## What is included

- PostgreSQL tables for leads, approved programs, conversations, document outcomes, and approval-gated outgoing messages.
- A Docker stack for n8n and PostgreSQL.
- Four importable n8n workflow blueprints in `n8n-workflows/`.
- An `.env.example` file. Copy it to `.env` locally and set real values. `.env` is ignored by Git.

## Start locally

1. Install Docker Desktop.
2. Copy `.env.example` to `.env`, then replace all placeholder values.
3. In this folder run `docker compose up -d`.
4. Open `http://localhost:5678`, create the n8n owner account, and import every JSON workflow from `n8n-workflows/`.
5. In n8n create credentials for PostgreSQL, OpenAI, and Meta WhatsApp. Attach them to the matching nodes and activate the workflows.

## Workflow safety model

Inbound messages and documents are saved immediately, but every student-facing reply or reminder is inserted with `pending_approval`. The approval webhook changes one specific message to `approved`; only then can the WhatsApp sender deliver it. The workflow must never send directly from the inbound or reminder paths.

## Required external setup

- A Meta WhatsApp Business app must send incoming messages to the n8n inbound webhook and supply an access token/phone number ID.
- OpenAI API billing and an API key are required for AI extraction/classification. The program-answer step must query the `programs` table first and must not invent data.
- A storage provider (S3, Supabase Storage, or Google Drive) is required if original test-document files are retained. Use only fake documents for this assignment.

## Deployment

Deploy this Docker stack to a server that supports persistent volumes and HTTPS (for example Railway, Render, DigitalOcean, or a VPS). Set the same environment variables in the host. GitHub Pages remains the public dashboard UI; it should call the public n8n webhooks only through authenticated staff actions.
