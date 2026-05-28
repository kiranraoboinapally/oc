# Installing Red Hat CodeReady Containers (CRC)

## Step 1: Download CRC

Download the latest Red Hat CodeReady Containers (CRC) package from the official Red Hat website.

Also download the Pull Secret required for OpenShift cluster setup.

Required Files:

* CRC Linux Package (`crc-linux-amd64.tar.xz`)
* OpenShift Pull Secret

---

# Step 2: Transfer Files to VM Server

Use WinSCP or any SCP tool to transfer the downloaded files from your local system to the Linux VM server.

Example Location:

```bash
/home/user/
```

---

# Step 3: Extract the CRC Package

Navigate to the directory where the CRC package is located and run:

```bash
tar -xvf crc-linux-amd64.tar.xz
```

This extracts the CRC installation directory.

---

# Step 4: Navigate to Extracted Directory

```bash
cd crc-linux-2.61.0-amd64
```

---

# Step 5: Move CRC Binary to System Path

Copy the CRC binary into `/usr/local/bin` so it can be executed globally.

```bash
sudo cp crc /usr/local/bin/
```

Verify installation:

```bash
crc version
```

---

# Step 6: Configure CRC

Run the setup command:

```bash
crc setup
```

During setup, provide the downloaded Pull Secret when prompted.

---

# System Requirements

Minimum recommended RAM:

* 11 GB to 12 GB RAM

If sufficient memory is not available, the `crc setup` command may fail.

Recommended Resources:

* 4 CPU cores
* 12 GB RAM
* 35 GB free disk space

---

# Reference Documentation

Official CRC Documentation:

[Red Hat CRC Documentation](https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/?utm_source=chatgpt.com)
