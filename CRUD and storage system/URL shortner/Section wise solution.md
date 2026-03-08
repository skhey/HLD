# Functional Requirements – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about functional requirements for a URL shortening service.

---

## Core Features
**Question:** *What are the absolutely essential features the system must support?*

**Interviewer:** The two core features are:
1.  **Shorten URL:** Given a long URL, the system must generate a unique, short alias (e.g., `https://short.url/abc123`).
2.  **Redirect:** When a user accesses the short URL, they must be redirected (HTTP redirect) to the original long URL.

Everything else is secondary for this design session.

---

## Optional Features
**Question:** *Are there nice‑to‑have features we might consider (e.g., user accounts, analytics, customisation)?*

**Interviewer:** Yes, we can consider the following as optional enhancements, but they are not mandatory for the core design. We can mention them, but focus on the core first:
- **Custom aliases:** Allow users to specify their own short path (e.g., `bit.ly/mylink`).
- **Expiration:** Allow URLs to expire after a certain time (TTL).
- **Basic analytics:** Track the number of clicks per short URL.
- **User accounts:** Users can create an account to manage their links and view analytics.

For now, let's focus on the core. If we have time, we can discuss how we might add one of these.

---

## User Roles
**Question:** *Are there different types of users (e.g., regular users, admins)? Do they have different permissions?*

**Interviewer:** Not applicable. For this design, we are building a public service. There are no user roles or permissions. Anyone can shorten a URL without logging in.

---

## Input/Output
**Question:** *What data does the system accept? What does it return? Are there any size or format constraints?*

**Interviewer:**
- **Input:** The system accepts a long URL (a string). We should validate that it's a properly formatted URL.
- **Output:** The system returns the generated short URL (a string).
- **Constraints:** Long URLs can be up to, say, 2048 characters (common limit). Short URLs should be as short as possible, ideally 6–7 characters.

---

## Actions/Workflows
**Question:** *What are the primary user workflows (e.g., create, read, update, delete)?*

**Interviewer:** There are only two primary workflows:
1.  **Create (Shorten):** User provides a long URL → System returns a short URL.
2.  **Read (Redirect):** User visits a short URL → System redirects to the long URL.

We do not need to support update or delete operations for this design. (If we add user accounts later, we might, but for now, it's not required.)

---

## Out of Scope
**Question:** *What features should we explicitly not design for now?*

**Interviewer:** We will explicitly **not** design the following:
- User accounts and authentication.
- Detailed analytics dashboards (though we might mention how we'd collect click data).
- Link management (editing, deleting).
- Admin interfaces or moderation tools.
- Billing or payment integration.
- Support for custom domains.

We are building a simple, public URL shortener with just the core shortening and redirection features.

---

## Summary for the Candidate

Based on these answers, you now know:
- **Core:** Shorten + Redirect.
- **Optional:** Custom aliases, expiration, basic analytics (nice to mention).
- **Scale:** Not given yet – you'll need to ask about traffic estimates next.
- **Constraints:** URL length up to ~2000 chars; short code length 6–7 chars.
- **Out of scope:** User accounts, management, admin features.

This gives you a clear scope to start your design. You can now move on to **non‑functional requirements** (scale, latency, availability) and **traffic estimation**.
