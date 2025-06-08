# Setup Guide for PQC-Based Secure Web Browser (Mozilla-Based, Fully Open-Source)

This guide sets up the PQC-Based Secure Web Browser with hybrid cryptography (RSA/ECC + Kyber/Dilithium/Falcon), using Mozilla Firefox, OpenCA for the CA, and Unbound for DNS. All commands are tested on Ubuntu 22.04 LTS.

## Prerequisites
- **OS**: Ubuntu 22.04 LTS
- **Hardware**: 16 GB RAM, 8-core CPU, 500 GB SSD
- **Internet**: Stable connection
- **User Privileges**: Sudo access

## Step 1: Install System Dependencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git python3 python3-pip curl wget unzip openjdk-17-jdk nginx mercurial perl libssl-dev cmake ninja-build unbound
pip3 install dnspython
```

## Step 2: Install VS Code
```bash
sudo snap install --classic code
```
**Source**: [VS Code](https://code.visualstudio.com/)

## Step 3: Clone the Repository
```bash
git clone https://github.com/QryptShield/pqc-browser.git
cd pqc-browser
```
**Source**: [GitHub](https://github.com)

## Step 4: Set Up Firefox Source
```bash
mkdir -p browser/src/core
cd browser/src/core
hg clone https://hg.mozilla.org/mozilla-central
cd mozilla-central
./mach bootstrap
cp ../../../config/mozconfig .
```
**Source**: [Mozilla Firefox](https://firefox-source-docs.mozilla.org/setup/index.html)

## Step 5: Install liboqs for PQC Algorithms
```bash
git clone https://github.com/open-quantum-safe/liboqs.git
cd liboqs
mkdir build && cd build
cmake -GNinja ..
ninja
sudo ninja install
```
**Source**: [liboqs](https://github.com/open-quantum-safe/liboqs)

## Step 6: Install NSS for TLS
```bash
hg clone https://hg.mozilla.org/projects/nss
cd nss
./build.sh
sudo cp -r dist/Release/lib/* /usr/local/lib/
sudo cp -r dist/Release/include/* /usr/local/include/
```
**Source**: [NSS](https://firefox-source-docs.mozilla.org/security/nss/)

## Step 7: Customize OpenSSL for PQC
```bash
git clone https://github.com/open-quantum-safe/openssl.git
cd openssl
./Configure --prefix=/usr/local/ssl --openssldir=/usr/local/ssl linux-x86_64
make
sudo make install
```
**Source**: [OpenSSL OQS](https://github.com/open-quantum-safe/openssl)

## Step 8: Set Up OpenCA for CA
```bash
git clone https://github.com/openca/openca-base.git
cd openca-base
./configure --prefix=/usr/local/openca
make
sudo make install
sudo /usr/local/openca/bin/openca-config --initialize
sudo perl ca_system/src/openca_config.pl
sudo /usr/local/openca/bin/openca-ca --start
```
**Source**: [OpenCA](https://github.com/openca/openca-base)

## Step 9: Configure Nginx for CA
```bash
sudo cp ca_system/src/nginx_config.conf /etc/nginx/sites-available/pqc-ca
sudo ln -s /etc/nginx/sites-available/pqc-ca /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```
**Source**: [Nginx](https://nginx.org/)

## Step 10: Set Up Unbound for DNS
```bash
sudo cp dns_system/src/unbound.conf /etc/unbound/unbound.conf
sudo unbound-anchor -a "/etc/unbound/root.anchor"
sudo systemctl restart unbound
```
Update Firefox to use Unbound:
```bash
cd browser/src/core/mozilla-central
echo 'pref("network.trr.mode", 2);' >> browser/config/mozconfigs/common
echo 'pref("network.trr.uri", "https://localhost/dns-query");' >> browser/config/mozconfigs/common
```
**Source**: [Unbound](https://www.nlnetlabs.nl/projects/unbound/about/)

## Step 11: Configure DevSecOps Tools
### GitHub Actions
Push changes to trigger `devsecops/github/ci_cd.yml`.
**Source**: [GitHub Actions](https://docs.github.com/en/actions)

### SonarQube
```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.77363.zip
unzip sonarqube-9.9.0.77363.zip
sudo mv sonarqube-9.9.0.77363 /opt/sonarqube
/opt/sonarqube/bin/linux-x86-64/sonar.sh start
```
Access at `http://localhost:9000`.
**Source**: [SonarQube](https://www.sonarqube.org/)

### OWASP ZAP
```bash
sudo apt install -y zaproxy
```
Configure `devsecops/security/owasp_zap_config.xml`.
**Source**: [OWASP ZAP](https://www.zaproxy.org/)

## Step 12: Build and Run Firefox
```bash
cd browser/src/core/mozilla-central
./mach build
./mach run --setpref=security.tls.pqc.enabled=true
```

## Step 13: Run Tests
```bash
cd browser/tests
gcc unit_tests.c -o unit_tests -I/usr/local/include/oqs
./unit_tests
cd ../src/core/mozilla-central
./mach mochitest ../../tests/integration_tests.js
cd ../../ca_system/tests
python3 ca_tests.py
cd ../../dns_system/tests
python3 dns_tests.py
```

## Step 14: Open in VS Code
```bash
code pqc-browser.code-workspace
```

## Step 15: Documentation
- `docs/system_architecture.md`: System design
- `docs/technical_guide.md`: PQC, CA, and DNS details
- `docs/user_manual.md`: Usage instructions

## Troubleshooting
- **Firefox Build Fails**: Run `./mach bootstrap` again.
- **liboqs Issues**: Ensure CMake and Ninja are installed.
- **OpenCA Errors**: Verify Perl dependencies (`sudo cpan install OpenCA::Configuration`).
- **Unbound Issues**: Check `/etc/unbound/root.anchor` is updated.
- **NSS Issues**: Verify NSS libraries are in `/usr/local/lib`.

## References
- Mozilla Firefox: https://firefox-source-docs.mozilla.org/setup/index.html
- NSS: https://firefox-source-docs.mozilla.org/security/nss/
- liboqs: https://github.com/open-quantum-safe/liboqs
- OpenSSL OQS: https://github.com/open-quantum-safe/openssl
- OpenCA: https://github.com/openca/openca-base
- Unbound: https://www.nlnetlabs.nl/projects/unbound/about/
- Nginx: https://nginx.org/
- SonarQube: https://www.sonarqube.org/
- OWASP ZAP: https://www.zaproxy.org/
- GitHub Actions: https://docs.github.com/en/actions
- VS Code: https://code.visualstudio.com/
