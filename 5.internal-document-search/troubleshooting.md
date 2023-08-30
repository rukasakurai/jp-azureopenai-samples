
# 
## Code
```
database = CosmosClient(endpoint, credential).get_database_client(database_name)
```
## Error
```
Exception has occurred: TypeError
can't concat str to bytes
  File "/home/rusakura/code/github.com/Azure-Samples/jp-azureopenai-samples/5.internal-document-search/src/backend/approaches/chatlogging.py", line 20, in <module>
    database = CosmosClient(endpoint, credential).get_database_client(database_name)
  File "/home/rusakura/code/github.com/Azure-Samples/jp-azureopenai-samples/5.internal-document-search/src/backend/app.py", line 13, in <module>
    from approaches.chatlogging import get_user_name, write_error
TypeError: can't concat str to bytes
```
## Issue
`endpoint` and `database_name` not being read from the `os.environ.get`

## Solution
I'm assuming that the issue is that I'm running the Python code within an environment that is different from the Ubuntu bash command line that I am using, for example maybe a conda environment.

Running `source ../../.azure/20230824temp2/.env` did not solve the issue, nor did VS Code > Command Palette > Python: Restart Language Server.

Not code to be committed to the repository, but the below worked
```
from dotenv import load_dotenv

load_dotenv("../../.azure/20230824temp2/.env")
```
