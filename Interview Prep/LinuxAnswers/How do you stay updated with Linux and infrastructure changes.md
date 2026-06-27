How do you stay updated with Linux and infrastructure changes?


Acronym: A V C V P V D
A = Advisory
V = Vulnerability Scan
C = Change Management
V = Validate (Dev/UAT)
P = Patch (Rolling)
V = Verify (Health/Logs)
D = Document


To keep our Linux systems and infrastructure up to date, we follow a scheduled maintenance window for patching, which ensures the environment is regularly updated with the latest security and bug fixes. Our security team shares vulnerability scan reports that identify critical CVEs and security issues. Based on those reports, we plan and schedule patching through the change management process during the maintenance window.

We also follow Red Hat security advisories and release notes to stay informed about new kernel updates, security patches, and bug fixes. Before applying patches to production, we validate them in non-production environments whenever required.

During production patching, we use a rolling patching approach to minimize or avoid downtime. Once the patching is complete, we verify system health by checking application services, monitoring dashboards, alerts, and logs to ensure everything is functioning as expected before closing the change."
