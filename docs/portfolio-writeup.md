# Job Fit Analyzer: an n8n + Claude automation

*Portfolio write-up for danibytes.dev*

## The problem

Job hunting means reading a lot of postings and guessing which ones fit. I wanted a faster, more honest first pass: paste a posting in, and get back a score, the skills that match, the gaps, and any red flags. The catch is that an AI will happily say you are qualified for everything. I needed it to judge me only on skills I can actually back up.

## What I built

A workflow in n8n, an open-source automation tool, that runs like this:

1. **Form:** I paste a job title, company, link and description.
2. **Claude API:** The posting goes to Claude with a system prompt listing my confirmed skills, the skills I am still learning (treated as gaps), a scoring rubric, and what counts as a red flag.
3. **Validation:** A short JavaScript node checks that the reply is valid JSON with the fields I expect.
4. **Results:** If it is valid, a row is added to a Google Sheet and I get a plain-text summary email.
5. **Failure path:** If it is not valid, I get an alert email with the raw reply, and nothing is written to the Sheet.

The Claude call retries up to three times, and a Sheets problem does not stop the email from sending.

I tested it on real Indeed postings: a junior web developer role, an eCommerce front-end role, and an Android engineering role that correctly scored low (15/100) because I have no confirmed Android experience. I also forced the failure path on purpose to confirm the alert email arrives.

## What I learned

- **Tell the model what it may not claim.** Giving it a confirmed-skills list plus a not-confident-yet list kept the results honest. Gaps showed up as gaps.
- **Treat job descriptions as untrusted input.** The prompt tells Claude to ignore instructions inside the posting.
- **AI output needs validation.** My first version broke when the reply was cut off by the token limit, and later when the reply started with a thinking block instead of text. Reading the right block, raising the limit, and checking the JSON before using it fixed both.
- **Rubrics reduce score swings.** Early on, running a posting more than once gave scores that moved noticeably. I added a scoring rubric (what 0-30, 31-60, 61-80 and 81-100 mean, judged mainly on required skills) and the swings got smaller. They did not disappear, so I treat the score as a rough guide, not a verdict.
- **Debugging n8n:** Pinned test data makes a node skip running (my Sheet stopped updating because of one). Expressions need Expression mode, not Fixed. Test URLs only work while the workflow is listening.
- **Google OAuth is fiddly:** redirect URIs must match exactly, apps in testing mode expire, and you have to add yourself as a test user.
- **Plan for failure.** The alert path took about as long to build as the happy path, and it is the part I would trust most if I left this running.

## What is next

- Publish the workflow for always-on use (it currently runs on demand while n8n is running).
- Run the same posting several times to measure score consistency properly.
- Add a step that drafts a tailored resume from the confirmed-skills list, with me approving before anything is sent.

**Stack:** n8n, Claude API, Google Sheets, Gmail, JavaScript.
