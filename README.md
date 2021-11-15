
This is a dummy minimal structure for using ansible to deploy cloud 
applications, e.g. hosted on kubernetes or any other PaaS.

Due the "hosts" concept of ansible, it seems it is not suited as a config management tool for such 
"host-less" applications. But in the end it is just a semantically odd naming.

Just think about every occurence of "host" is just something like "target", e.g. a specific kubernetes cluster.

A minimal structure might look like this:

```
├── dummy_deploy.yml
├── group_vars
│   ├── all.yml
│   ├── my_app
│   │   └── common_configurations.yml
│   ├── my_app_dev
│   │   └── configs.yml
│   └── my_app_prod
│       └── configs.yml
├── host_vars
│   ├── kubernetes_experimental.yml
│   └── kubernetes_live.yml
├── my_app_dev
├── my_app_experimental
└── my_app_prod
```

We are organizing the different application systems (e.g. dev, staging, prod, testing1, etc..) in "inventories", 
so here the files in the root dir `my_app_dev`, `my_app_experimental`, `my_app_prod`. 

In this example we have two different applications configuartions organized in the group_vars folder called `my_app_dev`, `my_app_prod`. 

Additionally we have two "targets", in this case two differnt kuberbetes stubs organized in the "host_vars" folder 
called `kubernetes_experimental` and `kubernetes_live`. 

So now we can deploy all kind of permutations managed in inventory files. For example "deploy the prod configuration 
to the experimental kubernetes":

```
my_app:
    children:
        my_app_prod:
            hosts:
                kubernetes_experimental
```

The playbook is really just a stub here, doing nothing then showing how configurations can be organized for different
deploys:

```
- name: my app deploy
  hosts: my_app
  tasks:
    - name: Deploy to k8s
      ansible.builtin.debug:
        msg: Deploy {{app_name}} with {{app_configuration}} to {{k8s_api}}
```

All the different deploys then can be executed with this command:

```
ansible-playbook -i my_app_experimental dummy_deploy.yml
...
PLAY [my app deploy] ***********************************************************

TASK [Gathering Facts] *********************************************************
ok: [kubernetes_experimental]

TASK [Deploy to k8s] ***********************************************************
ok: [kubernetes_experimental] => {
    "changed": false,
    "msg": "Deploy MyApp with myapp_prod to http://experimentalk8s.com"
}

PLAY RECAP *********************************************************************
kubernetes_experimental    : ok=2    changed=0    unreachable=0    failed=0    s
```

Take note of the `all.yml` where all the boilerplate is absorbed to tell ansible not to use a remote ssh connection. 
Most likely the the deploy just just yaml over http requests.