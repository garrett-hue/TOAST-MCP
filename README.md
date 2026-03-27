[README.md](https://github.com/user-attachments/files/26316837/README.md)
# Toast MCP Server — CO04 7th & Colorado

MCP server that exposes Toast POS sales data for **CO04 7th & Colorado** (Anthony's Pizza & Pasta) over HTTP so **Zapier** (or any MCP client) can call it inside a Zap.

---

## Tools Available

| Tool | Description |
|------|-------------|
| `toast_get_sales_summary` | Net sales, gross sales, tax, tips, check count, avg check for a period |
| `toast_get_orders` | Paginated list of individual orders with totals and payment methods |
| `toast_get_payment_breakdown` | Sales grouped by payment type (CREDIT, CASH, GIFT_CARD, etc.) |
| `toast_get_top_items` | Top-selling menu items ranked by quantity sold |

All tools accept a `start_date` / `end_date` in ISO 8601 format and return either `json` (default) or `markdown`.

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `TOAST_CLIENT_ID` | ✅ | Toast API client ID |
| `TOAST_CLIENT_SECRET` | ✅ | Toast API client secret |
| `MCP_API_KEY` | Recommended | Bearer token that Zapier must send to protect the endpoint |
| `PORT` | Optional | HTTP port (default: `3000`) |

---

## Deployment (Render — recommended free tier)

1. Push this folder to a GitHub repo.
2. Create a new **Web Service** on [render.com](https://render.com).
3. Connect your GitHub repo.
4. Set:
   - **Build Command:** `npm install && npm run build`
   - **Start Command:** `npm start`
5. Add the environment variables above in Render's dashboard.
6. Deploy — Render gives you a public HTTPS URL like:
   `https://toast-co04-mcp-server.onrender.com`

Your MCP endpoint will be:
```
https://toast-co04-mcp-server.onrender.com/mcp
```

Health check:
```
https://toast-co04-mcp-server.onrender.com/health
```

---

## Connecting to Zapier

Zapier supports MCP servers natively. Inside a Zap:

1. Add a new **Action** step.
2. Search for **"MCP"** (or the Zapier MCP client app).
3. Set the **Server URL** to your deployed endpoint:
   `https://your-render-url.onrender.com/mcp`
4. If you set `MCP_API_KEY`, add the **Authorization header**:
   `Bearer your-mcp-api-key`
5. Choose a tool (e.g., `toast_get_sales_summary`).
6. Map the `start_date` and `end_date` fields from earlier Zap steps (e.g., a Formatter step that outputs today's date range).

### Example Zap: Daily Sales to Google Sheets

```
Trigger: Schedule (daily at 10pm)
  ↓
Step 1: Formatter → Date/Time → Today at 00:00 (ISO)
Step 2: Formatter → Date/Time → Today at 23:59 (ISO)
  ↓
Step 3: MCP → toast_get_sales_summary
         start_date: [Step 1 output]
         end_date:   [Step 2 output]
         response_format: json
  ↓
Step 4: Google Sheets → Create Row
         Date: today
         Net Sales: [net_sales from Step 3]
         Check Count: [check_count from Step 3]
         Avg Check: [avg_check from Step 3]
```

---

## Running Locally

```bash
export TOAST_CLIENT_ID=your_client_id
export TOAST_CLIENT_SECRET=your_client_secret
export MCP_API_KEY=choose-a-secure-key

npm run dev
# Server at http://localhost:3000/mcp
```

Test with curl:
```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer choose-a-secure-key" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "toast_get_sales_summary",
      "arguments": {
        "start_date": "2026-03-26T00:00:00.000-0600",
        "end_date": "2026-03-26T23:59:59.000-0600"
      }
    }
  }'
```

---

## Getting Toast API Credentials

1. Log into **Toast Web** → **Integrations** → **API Access**.
2. Create a new **API Client** with the scopes:
   - `orders:read`
   - `labor:read`
   - `menu:read`
3. Copy the **Client ID** and **Client Secret** into your environment variables.
