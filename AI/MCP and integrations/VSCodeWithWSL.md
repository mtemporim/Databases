# Instalação do WSL no Windows 11 e integração com o Visual Studio Code

## Passo 1 — WSL2 no Windows (do jeito “padrão Microsoft”)

### 1) Checar pré-requisito do Windows

Você precisa estar no **Windows 10 2004+ (Build 19041+)** ou **Windows 11** para usar o fluxo simples com `wsl --install`. ([Microsoft Learn][1])
(Para conferir rápido: `Win + R` → `winver`)

### 2) Instalar WSL (PowerShell como Admin)

Abra **PowerShell (Admin)** e rode:

```powershell
wsl --install
```

Esse comando habilita os recursos necessários e instala uma distro (por padrão Ubuntu). Depois, **reinicie**. ([Microsoft Learn][2])

> Se aparecer erro de virtualização, normalmente é **VT-x/AMD-V desativado no BIOS/UEFI** (ou Hyper-V/Virtual Machine Platform não habilitado). O `wsl --install` tenta habilitar o que precisa, mas a virtualização no BIOS depende da máquina.

### 3) Primeiro boot do Ubuntu

Depois do reboot:

* Abra **Ubuntu** no menu Iniciar
* Crie usuário e senha Linux (isso fica só no WSL)

### 4) Garantir que está em WSL2 (e não WSL1)

No PowerShell (normal, não precisa Admin), veja as distros e versões:

```powershell
wsl -l -v
```

Se a distro não estiver como **Version 2**, rode:

```powershell
wsl --set-default-version 2
wsl --set-version Ubuntu 2
```

O `--set-default-version 2` define WSL2 como padrão para novas distros. ([Microsoft Learn][3])

### 5) Atualizar o WSL (recomendado)

```powershell
wsl --update
wsl --shutdown
```

A atualização pelo `wsl --update` é o caminho recomendado nas docs atuais. ([Microsoft Learn][4])

---

## VS Code no modo WSL (o que interessa pro nosso objetivo)

### 6) Instalar a extensão WSL do VS Code

No VS Code, instale a extensão oficial **WSL** (Remote – WSL). Ela permite abrir um workspace “dentro” do Linux. ([Visual Studio Code][5])

### 7) Abrir seu projeto dentro do WSL

Você tem 2 jeitos bons:

**Opção A (recomendada):** Abra o VS Code → `Ctrl+Shift+P` → **“WSL: New Window”** → depois **Open Folder…** (dentro do filesystem Linux).

**Opção B:** No terminal do Ubuntu (WSL), vá até a pasta do projeto e rode:

```bash
code .
```

A extensão faz o VS Code “entrar” no WSL. ([Visual Studio Code][5])

### Dica de DBA (performance)

Para evitar lentidão (I/O), mantenha repositórios **dentro do Linux** (ex.: `/home/seuusuario/projetos`) em vez de trabalhar pesado em `/mnt/c/...`.

---

## Checklist do que tem que estar OK ao final do passo 1

* `wsl -l -v` mostrando sua distro como **Version 2**
* VS Code mostrando no canto (ou no Command Palette) que está em **WSL**
* Você conseguindo abrir um diretório Linux e editar arquivos normalmente

---

Se você me disser **qual Windows você está (10 ou 11)** e **qual distro você prefere (Ubuntu ok?)**, eu já emendo o **passo 2** do jeito mais direto: instalar o que a gente precisa dentro do WSL (git/curl/Node) e preparar o VS Code para o Codex/integrações.

[1]: https://learn.microsoft.com/pt-br/windows/wsl/install?utm_source=chatgpt.com "Como instalar o Linux no Windows com o WSL"
[2]: https://learn.microsoft.com/en-us/windows/wsl/install?utm_source=chatgpt.com "How to install Linux on Windows with WSL"
[3]: https://learn.microsoft.com/en-us/windows/wsl/basic-commands?utm_source=chatgpt.com "Basic commands for WSL"
[4]: https://learn.microsoft.com/en-us/windows/wsl/install-manual?utm_source=chatgpt.com "Manual installation steps for older versions of WSL"
[5]: https://code.visualstudio.com/docs/remote/wsl?utm_source=chatgpt.com "Developing in WSL"
