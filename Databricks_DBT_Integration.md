h2. 🧾 Conclusion: DBT Job Execution in Databricks

Databricks provides multiple patterns to execute DBT workflows, but only some are production-grade. Below is a classification of valid patterns, their purpose, and rationale for inclusion/exclusion.

h3. ✅ Production-Grade Patterns

|| Pattern || Execution || Trigger || Notes ||
| *P1 – Notebook `%sh` cell* | Runs `dbt run` in a notebook shell cell | Manual / Scheduled Workflow | Simple and useful for dev/test, but limited |
| *P2 – Python Wheel Task*   | Packaged DBT project as Python wheel    | Databricks Job / Workflow    | Recommended for production pipelines       |
| *P3 – Bash Script Task*    | DBT run via shell script in DBFS        | Databricks Job / Workflow    | Lightweight, script-friendly alternative   |
| *P4 – External Orchestrator → Job API* | External trigger to a defined Databricks Job | Airflow / CI/CD / Event Trigger | Enterprise orchestration and param support |

*Note:* All of the above *require a Databricks Job* to be production-grade.

h3. ⚠️ Technically Valid, but Not Production-Grade

|| Pattern || Why Not Production-Grade ||
| *Notebook Manual Execution* | No retries, no scheduling, no isolation, no monitoring |
| *One-off `%sh` command runs* | Cannot be reused, lacks observability and alerting |

h3. ❌ Omitted / Redundant Patterns

|| Pattern || Why Omitted ||
| *P5 – `databricks-cli` trigger* | Not a unique execution model – it's just a wrapper over the Job API |
| *P6 – Dockerized CI/CD DBT run* | Runs outside Databricks compute; violates constraint of being Databricks-native |
| *P4 duplicate of P2/P3* | External triggers (Pattern 4) only initiate a Databricks Job, not a new run mode |

h3. 🧠 Final Notes

* Databricks Jobs are *mandatory* for any scalable, monitored, and automated production DBT pipeline.
* The choice between *Python Wheel (P2)* and *Bash Task (P3)* depends on team maturity and packaging practices.
* External orchestrators (*P4*) only *trigger* jobs — they don’t replace them.

h3. 🔁 Final Summary: How DBT Jobs Are Triggered on Databricks

# The two ways to programmatically start a Databricks Job are:
## *Job API* and *databricks-cli*

# These Jobs can run DBT using:
## *Notebook tasks*, *Python Wheel tasks*, or *Shell (Bash) tasks*

# These jobs can be triggered from:
## *Databricks Workflows (manually or scheduled)*, *external orchestrators (Airflow, CI/CD)*, or *scripts using CLI/API*




h2. 🚀 Databricks Job Execution Report (Job API vs Databricks CLI)

----

h3. 1. Overview – Job API vs Databricks CLI

|| *Feature* || *Job API (REST 2.1)* || *databricks cli (wrapper)* ||
| **Interface** | Raw HTTPS endpoint | Command-line tool |
| **Invocation** | *POST* */api/2.1/jobs/run-now* | `databricks jobs run-now` |
| **Payload** | JSON request body | CLI flags or `--json-file` |
| **Auth** | PAT or Azure AD bearer token (header) | PAT stored via `databricks configure` |
| **Output** | JSON with `run_id` | Same, printed to stdout |
| **Typical use** | Airflow, CI/CD, Terraform, custom apps | Shell scripts, Jenkins, local testing |

----

h3. 2. Job API – Run a Job (Production Method)

*Endpoint*  
`POST https://<workspace-host>/api/2.1/jobs/run-now`

*Required headers*  
`Authorization: Bearer <TOKEN>`  
`Content-Type: application/json`

{code:json|title=Common payload examples}
// minimal
{ "job_id": 12345 }

// notebook task with widget params
{
  "job_id": 12345,
  "notebook_params": {
    "env": "prod",
    "schema": "sales"
  },
  "idempotency_token": "daily-prod-20250624"
}

// python-wheel task with named params
{
  "job_id": 67890,
  "python_named_params": {
    "run_type": "incremental",
    "target_schema": "finance"
  }
}
{code}

*Optional top-level fields:* `timeout_seconds`, `idempotency_token`

----

h3. 3. Equivalent *databricks cli* Calls

{code:bash|title=CLI examples}
# minimal
databricks jobs run-now --job-id 12345

# notebook params
databricks jobs run-now --job-id 12345 \
  --notebook-params '{"env":"prod","schema":"sales"}'

# python wheel params
databricks jobs run-now --job-id 67890 \
  --python-named-params '{"run_type":"incremental","target_schema":"finance"}'

# full JSON payload from file
databricks jobs run-now --json-file run_config.json
{code}

----

h3. 4. Quick-Reference Matrix – Parameter Objects

|| *Param object (send one)* || *Container type* || *Example fragment* || *Mandatory?* || *Task type that reads it* ||
| `notebook_params` | map\<string,string\> | `{ "env":"prod", "schema":"sales", "gparamA":"foo" }` | No | Notebook (`dbutils.widgets`) |
| `python_params` | array\<string\> | `[ "--env", "prod", "gparamB" ]` | No | Spark Python task (`sys.argv`) |
| `python_named_params` | map\<string,string\> | `{ "run_type":"full", "gparamC":"bar" }` | No | Python wheel task |
| `jar_params` | array\<string\> | `[ "com.acme.Main", "--env", "prod", "gparamD" ]` | No | Spark JAR task |
| `spark_submit_params` | array\<string\> | `[ "--class", "com.acme.Main", "gparamE" ]` | No | Spark Submit task |

*Rules*  
* Only **one** parameter object per call.  
* All values must be strings; total JSON ≤ 10 KB.  
* `job_id` is the single required top-level field.

----

h3. 5. Key Points

# Production pipelines **must** run inside a Databricks *Job* for retries, scheduling and logs.  
# Job API and *databricks cli* are two interfaces that launch the same job.  
# Choose the parameter object that matches the configured task type.  
# Use `idempotency_token` to deduplicate retries and avoid duplicate runs.
