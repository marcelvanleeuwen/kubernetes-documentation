# Install kubectl on management machine:
<br>

### MacOS:

This section describes how to install and configure kubectl on a macOS management machine.

```sh
brew install kubectl
```
```sh
kubectl version --client
```
<br>
Check which shell you are using:

```sh
echo $SHELL
```
<br>

If you use zsh:

```sh
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
source ~/.zshrc
```
<br>

Creates a k alias for kubectl.
<br>

```sh
echo 'alias k=kubectl' >> ~/.zshrc
source ~/.zshrc
```
<br>