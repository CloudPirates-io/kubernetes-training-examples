#######################################################################
# OPTIONAL INSTALLATION FOR AUTOCOMPLETION
# per default not possible on windows, so use linux subsystem and bash autocompletion
#######################################################################
 
# Enable Linux Subsystem
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
 
# Install Ubuntu 18.04
https://www.microsoft.com/store/apps/9N9TNGVNDL3Q
 
# Create autocompletion scripts
minikube.exe completion bash > ~/minikube-completion.bash
kubectl.exe completion bash > ~/kubectl-completion.bash
 
# Add to .bashrc
source ~/minikube-completion.bash
source ~/kubectl-completion.bash
alias minikube='minikube.exe'
alias kubectl='kubectl.exe'
alias oc='oc.exe'
alias minishift=‚minishift.exe'