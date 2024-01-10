# MIT OL Devops Oncall Handbook

## Responsibilities

Your primary responsibility when oncall is ensuring the operational safety of
MIT Open Learning infrastructure.

### Answering Pages and Slack Broadcasts

You should action any pages you receive using the [MIT OL Oncall
Runbook](https://github.com/mitodl/ol-infrastructure/blob/main/docs/runbooks/oncall.md)
as your primary resource. If the particular page you action isn't represented in
the runbook, after you have mitigated the page you should add a new entry in the
runbook with appropriate steps.

If none of the available resources are appropriate for mitigating this page,
you should feel free to escalate the page. You can do this by clicking the
"Escalate" button in OpsGenie.

If anyone uses the at SRE mechanism on Slack, you should answer their question
or help them with their issue. If the request isn't quick, you should likely
create a Github Issue and escalate to your team lead for planning purposes.

### Operational Excellence Improvements

As MIT OL oncall your secondary responsibility is to act as operational
excellence czar for the duration of your shift during business hours.

This involves creating Github issues where gaps exist in the operational safety
of our infrastructure and advocating for existing issues to be resourced,
working on them yourself where appropriate.

This generally takes priority over your regular sprint work for the duration of your
shift.

### Incident Response and "Clarification of Error" Process

As MIT OL oncall you will likely be the first responder in the event of customer
facing infrastructure outages.

Obviously, your first job is to extinguish the flames and end the mitigate the
cause of the outage.

However, once the heat of the moment has passed, if the outage was siginficant
(For example, lasted longer than 10 minutes) then it probably makes sense to
leverage the COE process in order to help us improve and grow so we can better
serve our customers.

You can find the [COE Template
Document](https://docs.google.com/document/d/1Hdpb1mLzl8f9OuQziU10cb2DKtTXgBE_a-2u8h1IrRs/edit?usp=sharing)
in Google docs is a great starting point. Feel free to make a copy of this doc
and fill it out. The template will prompt you to collect a bunch of data on the
incident and answer a number of questions about the outage.

Be sure to do the data gathering piece yourself immediately after the outage
concludes or shortly thereafter, and then, either call a team meeting or use one
with an open agenda (like our bi-weekly check-in) to present your COE and
collaborate on asking and answering the remaining questions.

