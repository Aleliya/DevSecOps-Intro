# Lab 6

## Task 1

### Terraform Tool Comparison

**Terraform Scanning Results:**

- **tfsec**: 53 findings
- **Checkov**: 78 findings
- **Terrascan**: 22 findings

### Pulumi Security Analysis:

**Pulumi Scanning Results (KICS)**: 6 findings
- **HIGH severity**: 2
- **MEDIUM severity**: 2
- **LOW severity**: 0

### Terraform vs. Pulumi:

**Terraform (HCL - declarative):**

- Easier for static analysis
- Predictable resource structure
- The security of the state file is critical
- Secrets require external solutions

**Pulumi (YAML - programmatic):**

- Difficult to analyze because of the dynamic logic
- Built-in state encryption
- Best work with secrets
- Complex dependencies are possible

### KICS Pulumi Support:

**Successfully detected KICS:**

- S3 Bucket Security Issues
- Security Group Misconfigurations
- RDS Security Issues
- IAM Policy Issues
- Secrets in Configuration

**Gaps in KICS detection:**

- Secrets in User Data
- EBS Volume Encryption
- CloudWatch Log Issues
- EKS Cluster Security
- Secrets in Outputs

### Critical Findings:

1. Misconfigured S3 buckets can leak private information to the entire internet or allow unauthorized data tampering / deletion
2. Ensure S3 buckets do not have, a both public ACL on the bucket and a public access block.
3. RDS Instance Auto Minor Version Upgrade flag disabled.
4. RDS Instance publicly_accessible flag is true
5. Security Groups - Unrestricted Specific Ports - (SSH,22)

### Tool Strengths:

**tfsec Strengths:**

- **Terraform-specific patterns**: Deep understanding of Terraform HCL
- **AWS best practices**: Comprehensive AWS security checks
- **Fast feedback**: Quick scan times ideal for developer workflow
- **Low false positives**: Accurate results with good precision

**Checkov Strengths:**

- **Multi-cloud coverage**: AWS, Azure, GCP, Kubernetes support
- **Compliance frameworks**: CIS, PCI-DSS, HIPAA checks
- **Custom policies**: Extensive customization capabilities
- **Infrastructure breadth**: Largest rule set (700+ policies)

**Terrascan Strengths**:

- **Compliance focus**: Strong regulatory requirement coverage
- **Policy accuracy**: Well-tested policies with good documentation
- **Enterprise features**: Integration with Tenable ecosystem
- **Resource relationships**: Understanding resource dependencies

**KICS Pulumi Strengths:**

- Multi-format support: Terraform, CloudFormation, Pulumi, Ansible, Docker
- Pulumi specialization: Best Pulumi support among scanners
- Open source focus: Community-driven development
- CI/CD integration: Easy pipeline integration

## Task 2 

### Ansible Security Issues 

**KICS Ansible findings**: 9 findings
- **HIGH severity**: 8
- **MEDIUM severity**: 0
- **LOW severity**: 1

### Best Practice Violations

1. **RedundantAttribute** - Hardcoded secret key should not appear in source
2. **IncorrectValue** - State's task when installing a package should not be defined as 'latest' or should have set 'update_only' to 'true'
3. **RedundantAttribute** - Hardcoded secret key should not appear in source

### KICS Ansible Queries and Remediation Steps:

1. **Permissive sudo configuration** - Prevent allowing passwordless sudo for all commands!

Remediation Code:

```bash
# SECURE: Configure restricted sudo access
- name: Configure secure sudo for app user
  copy:
    content: |
      # Application user sudo permissions
      appuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp
      appuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl status myapp
      appuser ALL=(ALL) NOPASSWD: /usr/bin/logrotate /etc/logrotate.d/myapp
      # Specific commands only, require password for others
      appuser ALL=(ALL) ALL
    dest: /etc/sudoers.d/appuser
    mode: '0440'
    validate: '/usr/sbin/visudo -cf %s'

# Alternative: Use ansible built-in module
- name: Configure sudo using ansible module
  user:
    name: appuser
    groups: wheel
    append: yes

- name: Configure sudoers with password requirement
  lineinfile:
    path: /etc/sudoers
    line: '%wheel ALL=(ALL) ALL'
    validate: '/usr/sbin/visudo -cf %s'
```

2. **Installing unnecessary packages** - Development tools should not be used on production server!

Remediation Code:

```bash
# SECURE: Install only required production packages
- name: Install production packages only
  apt:
    name:
      - nginx
      - certbot
      - fail2ban
      - logrotate
    state: present
    update_cache: yes

# If debugging tools are absolutely needed, restrict access
- name: Install minimal debugging tools with restrictions
  apt:
    name:
      - tcpdump
    state: present
  when: ansible_env.DEBUG_MODE | default(false)

- name: Set capabilities for tcpdump (non-root usage)
  command: setcap 'CAP_NET_RAW+eip CAP_NET_ADMIN+eip' /usr/sbin/tcpdump
  when: ansible_env.DEBUG_MODE | default(false)

# Remove development packages if present
- name: Remove development packages
  apt:
    name:
      - build-essential
      - gcc
      - g++
      - gdb
    state: absent
    purge: yes
```

3. **Exposing application on all interfaces** - Should bind to specific interface or localhost.

Remediation Code:

```bash
# SECURE: Bind to localhost only
- name: Configure application to listen on localhost only
  lineinfile:
    path: /etc/myapp/config.yml
    regexp: '^listen:'
    line: 'listen: 127.0.0.1:8080'
    backup: yes

# Or use template with secure defaults
- name: Deploy secure application configuration
  template:
    src: secure_app_config.j2
    dest: /etc/myapp/config.yml
    mode: '0640'
    owner: appuser
    group: appuser
    backup: yes
  notify: restart application

# With firewall configuration
- name: Configure firewall to restrict access
  ufw:
    rule: allow
    port: '8080'
    src: '10.0.0.0/8'  # Only internal network
    state: enabled

# Alternative: Use reverse proxy
- name: Configure nginx as reverse proxy
  template:
    src: nginx_proxy.j2
    dest: /etc/nginx/sites-available/myapp
    mode: '0644'
    backup: yes
```

4. **Fetching files without encryption** - Prevent transferring sensitive config in plaintext!

Remediation Code:

```bash
# SECURE: Encrypt files before transfer
- name: Encrypt configuration before backup
  command: gpg --batch --yes --passphrase "{{ backup_encryption_key }}" --symmetric --cipher-algo AES256 /etc/myapp/config.env
  args:
    creates: /etc/myapp/config.env.gpg

- name: Fetch encrypted configuration
  fetch:
    src: /etc/myapp/config.env.gpg
    dest: "{{ backup_dir }}/{{ inventory_hostname }}-config.env.gpg"
    flat: yes

# Or use ansible-vault for sensitive data
- name: Create encrypted backup with ansible-vault
  slurp:
    src: /etc/myapp/config.env
  register: config_content

- name: Save encrypted backup locally
  copy:
    content: "{{ config_content.content | b64decode | ansible.vault.encrypt(vault_secret) }}"
    dest: "{{ backup_dir }}/{{ inventory_hostname }}-config.env.vault"
    mode: '0600'

# Alternative: Use secure copy with SSH encryption only
- name: Secure fetch with validation
  fetch:
    src: /etc/myapp/config.env
    dest: "{{ backup_dir }}/"
    flat: yes
    validate: 'test -s %s && file %s | grep -q "text"'
```

5. **No checksum validation for templates** - Need to backup and validation before deployment.

Remediation Code:

```bash
# SECURE: Template deployment with validation and backup
- name: Create backup of current configuration
  copy:
    src: /etc/nginx/sites-available/app.conf
    dest: "/etc/nginx/sites-available/app.conf.backup-{{ ansible_date_time.epoch }}"
    remote_src: yes
    mode: '0644'
  when: ansible_check_mode | default(false) == false

- name: Deploy configuration template with validation
  template:
    src: app.conf.j2
    dest: /etc/nginx/sites-available/app.conf
    mode: '0644'
    backup: yes
    validate: '/usr/sbin/nginx -t -c %s'

- name: Verify configuration checksum
  stat:
    path: /etc/nginx/sites-available/app.conf
  register: config_stat

- name: Validate configuration syntax
  command: nginx -t
  register: nginx_validation
  changed_when: false
  failed_when: nginx_validation.rc != 0

- name: Rollback if validation fails
  copy:
    src: "/etc/nginx/sites-available/app.conf.backup-{{ ansible_date_time.epoch }}"
    dest: /etc/nginx/sites-available/app.conf
    remote_src: yes
    mode: '0644'
  when: nginx_validation.rc != 0

# Enhanced version with pre-deployment checks
- name: Check current config checksum
  stat:
    path: /etc/nginx/sites-available/app.conf
  register: current_config
  changed_when: false

- name: Deploy only if template changed
  template:
    src: app.conf.j2
    dest: /etc/nginx/sites-available/app.conf
    mode: '0644'
    backup: yes
    validate: '/usr/sbin/nginx -t -c %s'
  when: current_config.stat.checksum != (lookup('template', 'app.conf.j2') | hash('sha256'))
```

etc.

---

## Task 3

### Tool Comparison Matrix:

| **Criterion**              | **tfsec** | **Checkov** | **Terrascan** | **KICS (Pulumi + Ansible)** |
|-----------------------------|-----------:|-------------:|---------------:|-----------------------------:|
| **Total Findings**          | 53 | 78 | 22 | 15 (Pulumi 6 + Ansible 9) |
| **Scan Speed**              | Fast | Medium | Medium | Slow |
| **False Positives**         | Low | Medium | Medium | High |
| **Report Quality**          | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Ease of Use**             | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| **Documentation**           | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **Platform Support**        | Terraform only | Multiple  | Multiple  | Multiple  |
| **Output Formats**          | JSON, text, SARIF | JSON, JUnit, CSV, SARIF | JSON, CSV, SARIF | JSON, SARIF, HTML |
| **CI/CD Integration**       | Easy | Easy | Medium | Medium |
| **Unique Strengths**        | Easy installation, high accuracy of Terraform scans | Large IaC coverage, flexible policies | Powerful Policy-as-Code engine, good integration with CI | Pulumi and Ansible support, a single engine for different IaCs |

### Category Analysis:

| **Security Category**        | **tfsec** | **Checkov** | **Terrascan** | **KICS (Pulumi)** | **KICS (Ansible)** | **Best Tool** |
|-------------------------------|-----------:|-------------:|---------------:|-------------------:|-------------------:|----------------|
| **Encryption Issues**         | 8 | 9 | 3 | 1 | 0 | Checkov |
| **Network Security**          | 9 | 17 | 6 | 0 | 0 | Checkov |
| **Secrets Management**        | 0 | 1 | 1 | 1 | 8 | KICS (Ansible) |
| **IAM / Permissions**         | 11 | 21 | 2 | 0 | 0 | Checkov |
| **Access Control**            | 10 | 10 | 3 | 1 | 0 | tfsec/Checkov |
| **Compliance / Best Practices** | 15 | 20 | 7 | 3 | 1 | Checkov |

### Lessons Learned:

**Key insights**:

- No single tool covers everything.
- False positives are inevitable, especially in complex IAM policies.
- The tools specialize: tfsec for speed, Checkov for completeness, Terrascan for compliance
- Pulumi security ecosystem provides only basic coverage
- Ansible security requires specialized checks, only good for secrets

### CI/CD Integration Strategy:

#### For small teams:

1. **Pre-commit:** `tfsec` — a quick feedback 
2. **PR Check:** `Checkov` — a basic security  
3. **Release:** `Checkov` + `Terrascan` — a full pre-release check

#### For enterprise

1. **Developer:** `tfsec` + `KICS` — a verification during local development 
2. **PR Gate:** `Checkov` + `Terrascan` — an utomatic pre-merge verification
3. **Release:** `Checkov` - a verification of compliance with security policies (Compliance validation) + manual approval (Security approval)

#### For mixed environments

1. **Terraform:** `Checkov` + `tfsec`  
2. **Pulumi:** `KICS` 
3. **Ansible:** `KICS`
4. **Kubernetes:** `Checkov`