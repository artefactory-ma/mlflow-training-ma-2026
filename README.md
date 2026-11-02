# MLflow, from experiment to production: the Telco churn lab

Companion lab of the Data Science team training *MLflow - from experiment to production*.
One notebook, one project: a churn classifier for **Telco** (a fictional client, using the public IBM Telco churn sample).
It goes from the first logged run to a registered, served and monitored model, then adds a bonus on LLMOps:
prompt registry, tracing, GenAI evaluation and AI Gateway.

## How it runs

The notebook runs in **Google Colab**. Everything it produces is stored in a remote **Databricks** workspace: runs, models, prompts, traces and data tables. This is close to real projects, where the code often runs outside the platform that stores the results (a laptop, a VM, a CI job).

| Option | How | When |
|---|---|---|
| **A. Colab + remote Databricks** *(the training)* | Colab runs the code; a personal access token connects it to Databricks | Code runs anywhere: laptop, VM, CI, Colab |
| B. Databricks Git folder | In Databricks: *Workspace > Create > Git folder* with this repo URL, then run on *Serverless* | The team works inside Databricks |
| C. Databricks import | In Databricks: *Home > Create > Folder*, *Import* the notebook, create a `data` folder and import the CSV into it | No Git access in the workspace |

The code is the same in all three. Inside Databricks (B, C), you are already authenticated: skip step 1.3 of the notebook.

## Option A, step by step

**1. Create a Databricks workspace.** Databricks Free Edition is enough and is free.

**2. Get the two connection items in Databricks.**
* **Workspace URL**: copy the browser address, up to the domain, for example `https://dbc-1a2b3c4d-5e6f.cloud.databricks.com`.
* **Personal access token**:
  1. Click your avatar (top right), then *Settings > User > Developer*.
  2. Next to *Access tokens*, click *Manage > Generate new token*.
  3. Copy the token. It starts with `dapi` and is shown only once.

**3. Open the notebook in Colab.**
* With GitHub: *File > Open notebook > GitHub*, then paste this repo URL.
* Without GitHub: *File > Upload notebook*, then pick `mlflow_telco_churn_lab.ipynb`.

**4. Add two Colab secrets.** Click the key icon in the left bar.
* `DATABRICKS_HOST`: the workspace URL.
* `DATABRICKS_TOKEN`: the token.
* Turn on *Notebook access* for both.

**5. Run the notebook from top to bottom.** Module 1 checks the connection and prints your Databricks user name.

**About the data.** The notebook looks for the data in three places, in this order:
1. `data/telco_churn.csv` in Colab's files. To use this, open the folder icon, create a `data` folder and upload the CSV from this repo.
2. A clone of this repo, if you set `REPO_URL` in cell 2.1. This also tags runs with the Git commit.
3. Otherwise, the public IBM URL.

Never paste the token in a cell. Secrets stay with your Google account and are not part of the notebook you share.

## What the lab creates in Databricks

| Where | What |
|---|---|
| Experiments `/Shared/telco/churn/*` | Training, evaluation and monitoring runs |
| Experiment `/Shared/telco/retention-assistant/development` | Prompts, traces and GenAI evaluations (bonus) |
| Unity Catalog `workspace.telco` | Volume `raw` (uploaded CSV files), table `customers` (data versions), model `telco_churn_classifier`, prompt `telco_retention_call_script`, table `call_list` |

To delete everything afterwards, set `CLEAN_UP = True` in the last cell.

## Using it in the training

* **On the deck:** a pink **▶ Demo next: notebook, module N** tag marks the last slide of each module.
* **In the notebook:** a **▶️ DEMO · Module N** banner marks where the demo starts and what to show.
* **Back to the deck:** a **⏸️ BACK TO SLIDES** banner marks where to stop and which slide comes next.

## Notes

* **No Spark in Colab.** The notebook creates and reads Delta tables by sending SQL to the workspace's SQL warehouse. Free Edition includes one, the *Serverless Starter Warehouse*. Its first query can take about a minute while it starts.
* **Data versioning** uses Delta table versions:
  * version 0 is the empty table, version 1 is October, version 2 is November;
  * runs log the pinned version as their dataset source;
  * the deck also covers DVC, for files kept outside Databricks.
* **Splits** are 60 / 20 / 20, stratified, with `random_state=42`. Tuning uses the validation set. The test set is used once, for the promotion decision.
* **Bonus LLM:**
  * the notebook tries a few Databricks Foundation Model endpoints;
  * if none answers, it uses an offline stand-in, so prompts, traces and evaluation still run;
  * edit `CANDIDATE_ENDPOINTS` to match the endpoints listed under *Serving* in your workspace.
* **scikit-learn is pinned below 1.9** (`>=1.6,<1.9`): version 1.9 changed the random forest slightly, and the pin gives everyone the scores shown on the slides.
* **MLflow 3.17 and later** save scikit-learn models with skops. Tree models are therefore registered with `skops_trusted_types=["sklearn.tree._tree.Tree"]`.
