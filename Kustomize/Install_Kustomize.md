# 📌 Installing Kustomize

Kustomize is a configuration management tool that simplifies Kubernetes manifest customization. This guide provides step-by-step instructions to install Kustomize on different operating systems.

---

## 🚀 Installation Steps

### **1️⃣ Install Kustomize on Linux & macOS**
```sh
curl -Lo ./kustomize "https://github.com/kubernetes-sigs/kustomize/releases/latest/download/kustomize_$(uname -s)_$(uname -m)"
chmod +x ./kustomize
mv ./kustomize /usr/local/bin/
```
Verify installation:
```sh
kustomize version
```

### **2️⃣ Install Kustomize on Windows**
```powershell
Invoke-WebRequest -Uri "https://github.com/kubernetes-sigs/kustomize/releases/latest/download/kustomize_windows_amd64.exe" -OutFile "kustomize.exe"
Move-Item -Path "kustomize.exe" -Destination "C:\Program Files\Kustomize\kustomize.exe"
[System.Environment]::SetEnvironmentVariable("Path", $Env:Path + ";C:\Program Files\Kustomize", [System.EnvironmentVariableTarget]::Machine)
```
Verify installation:
```powershell
kustomize version
```

---

## 📦 Installing Kustomize via Package Managers

### **Using Homebrew (macOS/Linux)**
```sh
brew install kustomize
```

### **Using Chocolatey (Windows)**
```powershell
choco install kustomize
```

### **Using Scoop (Windows)**
```powershell
scoop install kustomize
```

---

## 🛠 Verifying Installation
Run the following command to check if Kustomize is installed correctly:
```sh
kustomize version
```
Expected Output:
```sh
{kustomize/vX.Y.Z YYYY-MM-DDT00:00:00Z}
```

---

## 🚀 Next Steps
Now that Kustomize is installed, you can start using it to manage Kubernetes manifests!

- **Create a `kustomization.yaml` file**:
  ```sh
  mkdir k8s && cd k8s
  echo "resources: []" > kustomization.yaml
  ```

- **Apply a Kustomize configuration**:
  ```sh
  kubectl apply -k ./k8s
  ```

🔹 Happy Kustomizing! 🚀
