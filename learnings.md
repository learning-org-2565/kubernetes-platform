in this proejct already dependencies added other microsercvices , when we create new service it will resolve the dns name
✅ SUMMARY: YOUR QUESTIONS ANSWERED
QuestionAnswerDoes Sock Shop have dependencies?✅ YES! Front-end → Catalogue, Carts, Orders, User, etc.Where are they defined?In the application code, NOT in YAML filesHow do they connect?Via Kubernetes DNS (service names resolve to ClusterIPs)Will it work if I deploy all files?✅ YES! No modifications neededDo I need to add env vars?❌ NO, but you can ADD them later for learningIs this production-ready?⚠️ For learning: YES. For production: Add env vars


Why This Order?

Namespace must exist first
DBs are slow to start
Backend services wait for DB connections
Front-end talks to all backends