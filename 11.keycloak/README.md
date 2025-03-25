# Keycloak and Grafana Integration Guide

## Step 1: Install Keycloak on Kubernetes

### 1.1 Create Namespace for Keycloak
Execute the following command to create a namespace for Keycloak:

```bash
kubectl create ns keycloak
```

### 1.2 Deploy Keycloak
Deploy Keycloak using the provided YAML file:

```bash
kubectl create -f https://raw.githubusercontent.com/keycloak/keycloak-quickstarts/refs/heads/main/kubernetes/keycloak.yaml -n keycloak
```

### 1.3 Create an Ingress for Keycloak
Create an Ingress for Keycloak by entering the following command:

```bash
wget -q -O - https://raw.githubusercontent.com/keycloak/keycloak-quickstarts/refs/heads/main/kubernetes/keycloak-ingress.yaml | \
sed "s/KEYCLOAK_HOST/keycloak.$(keycloakInstanceExternalIp).nip.io/" | \
kubectl create -f -
```

### 1.4 Update Keycloak Instance Service External IP
Change your Keycloak instance service external IP as needed.

### 1.5 View Keycloak URLs
Enter the following command to see the Keycloak URLs:

```bash
KEYCLOAK_URL=https://keycloak.$(minikube ip).nip.io &&
echo "" &&
echo "Keycloak:                 $KEYCLOAK_URL" &&
echo "Keycloak Admin Console:   $KEYCLOAK_URL/admin" &&
echo "Keycloak Account Console: $KEYCLOAK_URL/realms/myrealm/account" &&
echo ""
```

You have successfully installed Keycloak on your Kubernetes cluster.

## Step 2: Configure Keycloak

### 2.1 Create a New Client in Keycloak
1. Log in to Keycloak Admin Console: `http://<keycloak-ip>:8080/admin`.
2. Select your realm (create a new one if needed).
3. Navigate to `Clients` → Click `Create`.
4. Enter the following details:
    - **Client ID:** grafana
    - **Client Protocol:** openid-connect
    - **Root URL:** `http://<grafana-ip>:3000`
    - **Valid Redirect URIs:** `http://<grafana-ip>:3000/login/generic_oauth`
5. Click `Save`.

### 2.2 Configure Client Settings
1. Go to `Clients` → Select `grafana`.
2. Under the `Settings` tab:
    - Enable `Standard Flow` → ON
    - Enable `Implicit Flow` → ON (Optional)
    - Set `Access Type` → confidential
3. Under the `Credentials` tab:
    - Copy the `Client Secret`.

### 2.3 Create a User for Grafana Login
1. Go to `Users` → Click `Create`.
2. Enter a username (e.g., `grafana-user`) → Click `Save`.
3. Go to `Credentials` → Set a password.
4. Assign a Role (optional, for role-based access in Grafana).

## Step 3: Configure Grafana for Keycloak Authentication

### 3.1 Update Grafana Configuration
Edit the Grafana configuration file (`/etc/grafana/grafana.ini` or `/usr/share/grafana/conf/defaults.ini`) with the following content:

```yaml
grafana.ini:
  server:
    root_url: http://10.111.0.34
  auth.generic_oauth:
    enabled: true
    name: Keycloak
    allow_sign_up: true
    client_id: grafana
    client_secret: $__file{/etc/secrets/grafana-client-secret/client-secret}
    scopes: openid email profile
    auth_url: http://10.111.0.52:8080/realms/grafana_realm/protocol/openid-connect/auth?prompt=login
    token_url: http://10.111.0.52:8080/realms/grafana_realm/protocol/openid-connect/token
    api_url: http://10.111.0.52:8080/realms/grafana_realm/protocol/openid-connect/userinfo
    role_attribute_path: contains(groups[*], 'Admin') && 'Admin' || 'Viewer'
    logout_url: http://10.111.0.52:8080/realms/grafana_realm/protocol/openid-connect/logout?redirect_uri=http://10.111.0.34/login

extraSecretMounts:
  - name: grafana-client-secret
    secretName: grafana-client-secret
    mountPath: /etc/secrets/grafana-client-secret
    readOnly: true
```

Replace `<keycloak-ip>` with your Keycloak server’s IP or domain.

### 3.2 Restart Grafana
Restart Grafana to apply the changes:

```bash
sudo systemctl restart grafana-server
```

## Step 4: Add Group Membership Mapper in Keycloak

### 4.1 Using the Default "profile" Scope
1. Click on the `profile` scope from your list (since it's marked as Default and contains basic profile information).
2. Look for a tab or section called `Mappers`.
3. Inside the `Mappers` section, click on `Add mapper` or `Create` button.
4. Select `Group Membership` from the list of available mapper types.
5. Configure the mapper with these settings:
    - **Name:** groups
    - **Token Claim Name:** groups
    - **Full group path:** OFF
    - **Add to ID token:** ON
    - **Add to access token:** ON
    - **Add to userinfo:** ON
6. Save the configuration.
7. Ensure this client scope is assigned to your Grafana client. Go back to `Clients`, select your Grafana client, and check if the `profile` scope is listed in the assigned scopes.

### 4.2 Creating a New Dedicated Client Scope for Groups
1. Go back to the `Client Scopes` main page.
2. Click `Create client scope`.
3. Name it `groups` or something similar.
4. Set `Protocol` to `OpenID Connect`.
5. Set `Type` to `Default` or `Optional` (Default is recommended).
6. Save it.
7. Add the `Group Membership` mapper to this new scope.
8. Assign this new scope to your Grafana client.

After making these changes, log out and log back into Grafana to get a new token with the group information included.

## Step 5: Test the Integration
1. Open Grafana: `http://<grafana-ip>:3000`.
2. Click `Sign in with Keycloak`.
3. Enter your Keycloak username and password.
4. You should be logged into Grafana using Keycloak authentication! 🚀

## Why Use Keycloak Instead of Grafana’s Built-in Authentication?

### ✅ Single Sign-On (SSO):
- Users don’t need separate credentials for Grafana. They can log in using their Keycloak account, which can be linked to LDAP, Google, GitHub, etc.

### ✅ Centralized User Management:
- Instead of managing users separately in Grafana, you can manage them centrally in Keycloak.
- When a user leaves the organization, revoking access in Keycloak removes access from Grafana and other linked services.

### ✅ Role-Based Access Control (RBAC):
- You can map Keycloak roles (Admin, Editor, Viewer) to Grafana roles dynamically.
- Example: Users with the "grafana_admin" role in Keycloak automatically get Admin access in Grafana.

### ✅ Multi-Tenant and Scalable:
- If you run multiple Grafana instances (for different teams or customers), you can manage access centrally with Keycloak.

### ✅ Security & Compliance:
- Enforce Multi-Factor Authentication (MFA) via Keycloak.
- Integrate with OAuth2, OpenID Connect (OIDC), and LDAP.
- Track logins and manage session timeouts in one place.

## When to Use Grafana's Built-in Authentication?
- ❌ If you only have a few users and don’t need SSO.
- ❌ If you don’t use other tools that need unified authentication.
- ❌ If Keycloak adds unnecessary complexity for a small setup.

However, in DevOps environments where you have multiple tools (Grafana, Prometheus, Kubernetes, Jenkins, etc.), using Keycloak for authentication simplifies access management across all of them.

## Final Thoughts
Using Keycloak for Grafana authentication is ideal when you want SSO, centralized user management, and security compliance across multiple services. But if you’re managing only a few Grafana users, the built-in authentication might be enough.

[Back to Main README](../README.md)