# AI Customer Support Agent (n8n)

An n8n workflow that reads customer emails from Gmail, drafts a reply using a company knowledge base, and sends it only after a human approves.

## Tools used

- n8n
- Gmail
- Pinecone (vector store for the knowledge base)
- Google Gemini (embeddings and reply generation)

## How it works

1. **Gmail Trigger** checks the inbox every minute for new emails.
2. **Edit Fields** extracts the sender, subject, message, and thread ID.
3. **Pinecone Vector Store** finds the relevant policy information from the knowledge base.
4. **Basic LLM Chain** (Google Gemini) writes a reply using only that information. If the answer is not available, it says so politely.
5. **Send message and wait for response** emails the proposed reply to a reviewer with an **Approve** button.
6. **If** checks whether the reply was approved.
7. **Send a message** sends the approved reply to the customer.

## Project structure

```
.
├── workflow.json        # n8n workflow
├── knowledge-base.md    # Company policies stored in Pinecone
├── README.md
└── screenshots/
    └── workflow.png
```

## Setup

1. **Import** `workflow.json` in n8n (Workflows → Import from File).
2. **Create credentials** in n8n:
   - Gmail OAuth2
   - Google Gemini (PaLM) API: get a key from [Google AI Studio](https://aistudio.google.com/apikey)
   - Pinecone API
3. **Reviewer email:** in the *Send message and wait for response* node, set **To** to the address that approves replies.
4. **Gmail Trigger filter:** under Filters → Search, use the following so the workflow ignores its own emails:
```
   -from:me -subject:"Approval Needed" in:inbox
```
5. **Knowledge base:** upload `knowledge-base.md` to the Pinecone index `customer-support-kb-v2` once, using the same Gemini embedding model as the workflow.
6. **Activate** the workflow.

## Testing

Send an email to the connected Gmail account from a different address, open the **Approval Needed** email, and click **Approve**. The reply is then sent to the original sender.

## Notes

- Replies are generated only from the knowledge base. Update `knowledge-base.md` and re-upload it when policies change.
- The workflow does not send anything to a customer without approval.
- Never commit API keys. n8n keeps them in its credential manager.
