# Infrastructure Runbook: Enterprise Cryptographic Software Integration (1C:HR, Kontur, Saby/SBIS)

## Project Overview
This repository contains a comprehensive infrastructure execution guide (Runbook) for deploying, updating, and configuring corporate electronic digital signatures (EDS/PKI) across a multi-company holding environment within a Windows/RDP infrastructure.

**The Problem:** An accounting operator required a deployment of newly issued digital signatures for executives across 4 distinct legal entities (*List of companies*). The newly issued high-security signatures from the Federal Tax Authority (*Example*) were provisioned as **non-exportable**, preventing standard key migration to the local Windows Registry.

**The Solution:** A hybrid cryptographic architecture was designed and executed:
1. Direct binding of hardware tokens (Rutoken/JaCarta) to the local Windows Cryptographic Service Provider (CSP).
2. Enterprise-level integration into **1C:Salary and Human Capital Management (v3.1)** with explicit session privilege isolation.
3. Web-auth flow optimization for **Kontur.Extern** and **Saby (SBIS)** platforms, allowing seamless pre-validation workflows without constant hardware token extraction.

---

## Tech Stack & Environment
* **Operating System:** Windows 10/11 Pro, Windows Server (RDP Terminal Infrastructure)
* **Cryptographic Engine:** CryptoPro CSP (v5.0 / 4.0), Kontur.Plugin web extension, 1C native `ExtraCryptoAPI` component
* **Business Applications:** 1C:Salary and Human Capital Management (Enterprise v3.1)
* **EDM & Reporting Ecosystems:** Kontur.Extern, Saby Online (SBIS)
* **Hardware Platforms:** Rutoken Lite / JaCarta (GOST R 34.10-2012)

---

## Deployment & Configuration Steps

### Step 1. Registering Non-Exportable Containers in Windows (CryptoPro CSP)
Since the keys are protected against extraction (throwing Error `0x8009000B`), a local registration mapping the certificate chain directly to the chip must be executed:
1. Insert the hardware token into the USB interface. Launch `CryptoPro CSP` -> Navigate to the `Service` tab.
2. Click **View Certificates in Container** -> Click **Browse**.
3. Select the required target organizational container (e.g., Executive / "Example" LLC, valid until 2027).
4. Click **Install**. If a certificate collision occurs, select *“Replace existing certificate with the new one, enforcing a direct link to the private hardware key”*.
5. Repeat the sequence for all executive hardware pairs.

### Step 2. Cryptographic Configuration in 1C:HR Enterprise
This configuration requires `Administrator` privileges within the 1C application platform to alter global database metadata:
1. Navigate to `Administration` -> `General Settings` -> Expand the `Digital Signature and Encryption` block.
2. Click the link **Certificates and Digital Signature/Encryption Applications**.
3. Click **Add** -> **For Signing and Encryption from Personal Store...**.
4. On the initial run, approve the automated installation of the 1C background cryptographic component (`ExtraCryptoAPI`).
5. For each imported certificate, map the mandatory ERP metadata attributes:
   * **Organization:** Map to the corresponding legal entity topology (e.g., LLC "Example").
   * **Individual:** Select the correct object from the `Individuals` directory to ensure accurate HR order signing metadata.
6. **Privilege & Session Isolation (Final 1C Stage):** Open the newly added certificate card. In the **Used By** field, replace the default `Administrator` profile with the specific target Accountant's end-user profile (e.g., *John Doe*). Click `Save and Close`.

### Step 3. Web Optimization & Reporting Node Configuration

#### Kontur Ecosystem (*List of Companies*)
1. Run the system diagnostics utility at `install.kontur.ru`.
2. Upgrade local background modules and browser extensions to the latest stable versions to mitigate browser hangs during hardware token polls. Restart the terminal workstation.
3. To enable document preparation without permanent hardware token connection, authenticate to the client cabinet using the executive's digital signature, go to security parameters, and enable **Login/Password Authentication** (assign profile to `example@email`). The hardware token will now be polled *only* at the final transaction stage when applying the cryptographic signature before sending data to federal regulatory endpoints.

#### Saby / SBIS Ecosystem (*Company*)
1. Authenticate to the web node `online.sbis.ru` using the newly registered executive certificate (valid until 2027).
2. Navigate to `Settings` -> `Our Companies` -> Open the profile for **"Example" LLC**.
3. Select the `Who Signs` / `Reporting Defaults` section.
4. Set the primary signing profile to the newly added certificate (valid until 2027), deprecating the legacy certificate profile expiring in 2026.

---

## Validation & Verification

Execute the following verification checklist to confirm infrastructure readiness:
- [x] **CryptoPro CSP:** The *Certificates* tab confirms valid root chains and reports status: *"Valid"*.
- [x] **1C:HR Application:** Logged in as the end-user accountant, the "1C-Reporting" dashboard shows active signing permissions for all 4 legal entities.
- [x] **Kontur.Extern:** Credential-based login functions without a token inserted; initiating document transmission triggers the correct secure prompt requesting hardware USB token authorization.
- [x] **Saby (SBIS):** The "Consulting" entity details display a green status indicator confirming active PKI mapping valid through 2027.
## Copyright and License
Copyright (c) 2026 zazauzr. All rights reserved.
This repository and all its contents (including documentation, scripts, and configuration files) are proprietary. Unauthorized copying, modification, distribution, or commercial use of any materials from this repository, via any medium, is strictly prohibited without the express prior written permission of the copyright holder.