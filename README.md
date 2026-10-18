Bangladesh Multi-Tool AI Agent 🇧🇩
A small LangChain AgentExecutor project for asking questions about Bangladeshi institutions, healthcare facilities and restaurants. It converts three Hugging Face datasets into SQLite databases, gives each one its own read-only tool, and uses a fourth tool for general web searches. Google Gemini decides which tool to call.
Quick start in Google Colab
Upload notebooks/Bangladesh_MultiTool_Agent.ipynb to Google Colab. Run the cells in order. Set GOOGLE_API_KEY in Colab Secrets (key icon), or type it into the notebook's hidden getpass() prompt when asked. Do not paste keys into notebook code or GitHub. The notebook contains the whole working code; the ZIP is not required in Colab.
Run locally
python -m venv .venv
source .venv/bin/activate             # Windows: .venv\Scripts\activate
pip install -r requirements.txt
export GOOGLE_API_KEY="your-key"     # Windows PowerShell: $env:GOOGLE_API_KEY="your-key"
python -m src.bd_agent
The initial run downloads the published datasets (internet required) and creates data/institutions.db, data/hospitals.db, and data/restaurants.db. Later runs reuse the databases; you can refresh them using prepare_databases(refresh=True) from Python. Gemini requests require a Google AI Studio API key, API access and quota. The public web-search tool requires connectivity and may occasionally be throttled.
Four LangChain tools
Tool	Answers
InstitutionsDBTool	School/college/madrasah and other institutional records; location, type, counts
HospitalsDBTool	Listed health facilities and hospitals; location, type, counts
RestaurantsDBTool	Restaurant records; available locations, types and other actual columns
WebSearchTool	General facts, cultural context, definitions, policies and current information


A database question is transformed into a SELECT query based on the loaded table's real schema, then executed against a read-only SQLite file, capped to 8 returned rows. Dangerous statements are blocked by the database authorizer. The agent then explains the result in natural language. Queries about missing fields (e.g. number of beds or ratings when not provided) should be answered as unsupported rather than guessed.
Dataset caveat: The dataset titled “all-bangladeshi-hospitals” also includes health offices, dispensaries and other institutions. A count of all rows is not necessarily a count of hospitals. Ask explicitly for type containing “Hospital” or specify your desired definition. These datasets are not live registers of availability, vacancies, bed capacity, or restaurant hours.

Example questions
- How many hospital-related facilities are listed in Dhaka district? → HospitalsDBTool
- How many schools or colleges are listed in Barisal? → InstitutionsDBTool
- Show some restaurants listed in Chattogram. → RestaurantsDBTool (if matching location columns exist)
- What does the Directorate General of Health Services do in Bangladesh? → WebSearchTool
Use verbose=True to show tool selection and intermediate steps. SQL generation uses one Gemini call and final routing/explanation uses LangChain's agent.
Dataset sources and acknowledgements
1. Institutional Information of Bangladesh
2. All Bangladeshi Hospitals
3. Bangladeshi Restaurant Data
Creator: Mahadih534 on Hugging Face. Dataset creators own their respective content. Check each dataset's licensing and attribution requirements before republishing any raw data. This GitHub package does not redistribute the datasets: it fetches them when the program runs. The institutional dataset currently lists an unknown license.
Repo structure
Bangladesh_MultiTool_Agent/
├── notebooks/Bangladesh_MultiTool_Agent.ipynb  # standalone Colab notebook
├── src/bd_agent.py                           # same logic for a local CLI
├── tests/test_agent.py                       # offline safety/schema tests
├── data/README.md                            # download instructions
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
Offline verification
python -m unittest discover -s tests -v
Tests create temporary sample tables and check the SQL constraints; they do not test Google's API or download the three live datasets. To try an offline count after downloading data, call offline_example().
Notes
- Model: gemini-3.8-flash by default; set GEMINI_MODEL to a different model supported by your account.
- langchain-classic is used intentionally because the assignment specifically requires AgentExecutor.
- Web search uses the public ddgs library, not a paid SerpAPI key. A failed search reports the failure rather than fabricating a result.
- Logs may include the user's question and retrieved records; avoid uploading personal information or API credentials.
