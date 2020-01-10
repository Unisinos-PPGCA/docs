# Minikube Setup

In order to setup minikube in our "edge" cluster an ansible script was created.
To execute it follow the steps below:

1. Edit the hosts and main.yaml files replacing the fields between `<>`.
   1. You can find the information in the private evernote file. If you have no access request to Cristiano.

2. Execute the command `ansible-playbook -i hosts main.yaml`