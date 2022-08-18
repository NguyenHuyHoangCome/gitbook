# WF Sample
This section describe sample sorce code of WF.  

---

## contents:

* [get some value from input menu in a task/not WF variables](#get-some-value-from-input-menu-in-a-tasknot-wf-variables)
* [pass variables from task_a to task_b](#pass-variables-from-taska-to-taskb)
* [create new user to linux ME](#create-new-user-to-linux-me)
* [get user list from linux ME and create selectable user list  (get user data inventory)](#get-user-list-from-linux-me-and-create-selectable-user-list-get-user-data-inventory)
* [get user from selectable user list](#get-user-from-selectable-user-list)
* [WF call to other WF](#wf-call-to-other-wf)

---
## Get some value from input menu in a task/not WF variables
For example, we get the input value menu. After that, print it went the task run successful.

Here is sample sorce code
```python
from msa_sdk.variables import Variables
from msa_sdk.msa_api import MSA_API

dev_var = Variables()
dev_var.add('A', var_type='Integer')
dev_var.add('B', var_type='Integer')

context = Variables.task_call(dev_var)
X = context['A']
Y = context['B']

Z = int(X) + int(Y)
context['C'] = Z

ret = MSA_API.process_content('ENDED', f'workflow initialized and ( X : {X}, Y: {Y})', context, True)
print(ret)
```
Debug the task:

Fill the input data 
![License01](./img/img10.jpg)

Run the task (task name is `Task A`) the task will show the input Variables A and B
![License02](./img/img11.jpg)

---
## Pass variables from task_a to task_b
In `task_A` we create the Variables name `C` and value is `A + B`.

Here is example code to show value `C`.
```python
from msa_sdk.variables import Variables
from msa_sdk.msa_api import MSA_API

context = Variables.task_call()
Z = context['C']

ret = MSA_API.process_content('ENDED', f'value get( C : {Z})', context, True)
print(ret)
```
Degub the task:

Here is result after user input 2 Variables`A`,`B` is `1` and `2`.

the `C`Variables is `A + B`
![License03](./img/img12.jpg)


---
## Create new user to linux ME
Example code 
```python
import json
from msa_sdk.variables import Variables
from msa_sdk.msa_api import MSA_API
from msa_sdk.order import Order

# List all the parameters required by the task
dev_var = Variables()
dev_var.add('device_id', var_type='Device')
dev_var.add('user.0.comment', var_type='String')
dev_var.add('user.0.group_id', var_type='String')
dev_var.add('user.0.login', var_type='String')
dev_var.add('user.0.home_dir', var_type='String')
dev_var.add('user.0.object_id', var_type='String')
dev_var.add('user.0.password', var_type='String')
dev_var.add('user.0.shell', var_type='String')
dev_var.add('user.0.user_id', var_type='String')


context = Variables.task_call(dev_var)

# read the ID of the selected managed entity
device_id = context['device_id']

# extract the database ID
devicelongid = device_id[3:]

# build the Microservice JSON params
object_parameters = {}

object_parameters['user'] = {}
for v in context['user']:
  object_parameters['user'][v['object_id']] = v


# call the CREATE for the specified MS for each device
order = Order(devicelongid)
order.command_execute('CREATE', object_parameters)

# convert dict object into json
content = json.loads(order.content)

# check if the response is OK
if order.response.ok:
    ret = MSA_API.process_content('ENDED',
                                  f'STATUS: {content["status"]}, \
                                    MESSAGE: successfull',
                                  context, True)
else:
    ret = MSA_API.process_content('FAILED',
                                  f'Import failed \
                                  - {order.content}',
                                  context, True)

print(ret)
```
Debug the task:

Chose the ME you want to create user
![License04](./img/img13.jpg)

Input the user profile

Then, click the run button
![License05](./img/img14.jpg)

The new user create sussessful in ME
![License06](./img/img15.jpg)




---
## Get user list from linux ME and create selectable user list  (get user data inventory)
Example code

```python
import json
from msa_sdk.variables import Variables
from msa_sdk.msa_api import MSA_API
from msa_sdk.order import Order

# List all the parameters required by the task
dev_var = Variables()

dev_var.add('device_id', var_type='Device')
# dev_var.add('drop')
context = Variables.task_call(dev_var)

# read the ID of the selected managed entity
device_id = context['device_id']

# extract the database ID
devicelongid = device_id[3:]

# build the Microservice JSON params

object_parameters = {}
object_parameters['user'] = '0';

# call the CREATE for the specified MS for each device
order = Order(devicelongid)
order.command_execute('IMPORT', object_parameters)

# convert dict object into json
content = json.loads(order.content)
hi = []
for i in json.loads(content['message'])['user']:
	h = {"user_id" : json.loads(content['message'])['user'][''+str(i)+'']['user_id'],
		 "password" : json.loads(content['message'])['user'][''+str(i)+'']['password'],
		 "object_id" : i
	}
	hi.append(h)
	
context['list'] = hi
# check if the response is OK

if order.response.ok:
    ret = MSA_API.process_content('ENDED',
                                  f'STATUS: {content["status"]}, \
                                    MESSAGE: successfull',
                                  context, True)
else:
    ret = MSA_API.process_content('FAILED',
                                  f'Import failed \
                                  - {order.content}',
                                  context, True)

print(ret)


```
Debug the task



---
## get user from selectable user list

---
## WF call to other WF
