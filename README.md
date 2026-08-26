# DuploCloud DevKit Workshop

## Overview

Welcome to the DuploCloud DevKit Workshop. This guide walks you through your pre-provisioned workshop environment — from first login all the way through building, deploying, and iterating on a DuploCloud extension.

By the end of this workshop you will have:

- Connected to a cloud-hosted Code Server environment running on AWS
- Launched Claude Code via DevKit and verified DuploCloud is operational
- Built and deployed a **SOC 2 Posture** extension using an AI-assisted workflow
- Run a posture assessment against your cloud environment
- Remediated security findings via CLI commands and Terraform pull requests
- Iteratively extended the SOC 2 Posture extension to add new capability

---

## Prerequisites

Before starting, confirm you have:

- Registered as a workshop attendee
- Received a **DuploCloud Workshops** invitation email containing your instance links and credentials

---

## Step 1: Connect to Your Workshop Instance

A workshop environment has been pre-provisioned in a DuploCloud-managed AWS account specifically for you. It includes an EC2 instance with Code Server already running, and DevKit configured to install automatically on first launch.

### Environment Permissions

The EC2 instance runs with an IAM instance role that has been scoped specifically for this workshop:

| Permission | Access |
|---|---|
| Amazon Bedrock | ✅ Full access (required for Claude Code and the AI agent) |
| AWS Read (IAM, EC2, S3, etc.) | ✅ Read-only (used by the SOC 2 Posture extension to audit your environment) |
| AWS Write | ❌ No write access via the instance role |

> **Note:** In this workshop environment, the DuploCloud agent uses the same IAM role and permissions as the instance role — so write access is unavailable through either path. In production deployments, DuploCloud supports assigning different roles and permission levels to meet your organization's requirements, allowing the agent to take actions (such as CLI-based remediations) while still routing everything through the human-in-the-loop approval flow.

### Connect

1. Open the workshop invitation email from **DuploCloud Workshops**.
2. Click **Open Your Workspace** — this takes you directly to Code Server.
3. Enter the password from the email when prompted.

> ⚠️ **Note:** The password is randomly generated for your session. Do not reuse it elsewhere.

---

## Step 2: Complete DevKit Setup

Once Code Server loads:

1. Wait a few seconds — a terminal pane will appear at the bottom of the screen showing the DevKit setup in progress.

The automated setup will:

- **Register DevKit** using the email address you used to sign up for the workshop
- Send a **license confirmation email** to that address — open it and click **Approve Request**, then return to the Code Server tab
- Automatically detect the approval and continue
- Set up **Docker** (images are pre-staged, so this is fast)
- Configure **Amazon Bedrock** via the DuploCloud account for LLM access
- Register the relevant **scopes** (AWS account and Kubernetes cluster)
- Download a pre-scaffolded SOC 2 extension from S3 to save time during the session

2. When you see **"Press any key to close terminal"**, hit any key to dismiss the pane.

---

## Step 3: Explore Code Server and Claude Code

Code Server comes pre-configured with **Claude Code**, connected to Amazon Bedrock using the Claude Sonnet model. You can access it via the button in the top-right corner of the interface, or directly from a terminal.

**To launch Claude from the terminal:**

1. Right-click in the grey area of the Code Server interface and choose **New Terminal**.
2. Type `claude` and press **Enter**.
3. A first-run setup wizard appears — press **Enter** to accept the defaults at each prompt.
4. When asked about **workspace access**, choose **Trust This Folder** — this is required for Claude Code to function correctly.

You are now at the Claude prompt and ready to start building.

---

## Step 4: Verify DuploCloud is Running

Before building anything, confirm that DuploCloud and the AI agent are working as expected.

1. Return to your workshop email and click **Open DuploCloud**.
2. Copy the password from the email and paste it into the login screen.
3. Your username is the email address the workshop invitation was sent to.
4. Click **Sign In**.

Once logged in:

1. Select the **LLM Agent** (one option is available).
2. Click **Select Scopes** — you will see two pre-configured scopes:
   - **AWS Account** — access to the workshop AWS environment
   - **Kubernetes Cluster** — access to the cluster running inside that account

**Run a quick validation test:**

1. Click on the **AWS Instance Role** scope.
2. Enter the prompt: `Show me the S3 buckets available in this account`
3. Click **Create Ticket**.
4. DuploCloud's human-in-the-loop approval flow will prompt you to review the command — ensure **Approve** is highlighted, then click **Submit**.
5. The agent will return a list of S3 buckets in the account.

✅ This confirms DuploCloud is running and the agent can communicate with the backend LLM.

> **Note:** Your workshop environment will remain available after the session ends, so don't worry about exploring every corner of DuploCloud right now — you'll have time to come back to it.

---

## Step 5: Build and Deploy the SOC 2 Posture Extension

Return to Code Server. Claude is ready at the prompt.

DevKit includes a skill called **`duplo-extension`** which guides you through building DuploCloud extensions — either from scratch via a prompted wizard, or by working with an existing scaffolded codebase. For this workshop, the SOC 2 Posture extension has been pre-scaffolded to ensure we can complete the build within the session time.

### Build Steps

1. At the Claude prompt, invoke the **`/duplo-extension`** skill.
2. Claude will scan the extensions directory, detect the pre-scaffolded extension, and confirm what it found.
3. When prompted, indicate that you want to **build and deploy the existing SOC 2 Posture extension**.
4. Press **Enter** to confirm — Claude is ready to proceed.

### Enable Auto Mode

To allow Claude to move through the build steps without requiring manual confirmation at each one:

- Hold **Shift** and press **Tab** three times.
- **Auto Mode** will appear at the bottom of the screen.

Claude will now automatically:

- Detect DuploCloud running locally via DevKit
- Identify the extension and compile it
- Deploy it to your DuploCloud instance

Watch the terminal output to follow along. The build and deployment takes a few minutes.

---

## Step 6: Run a SOC 2 Posture Assessment

Once the extension is deployed and visible in DuploCloud (this might require a hard refresh of your browser):

1. Navigate to the **SOC 2 Posture** extension inside DuploCloud.
2. Click **New Assessment**.
3. Select your target region (e.g., **US West 2**).
4. In the resource tag filter, specify a Tag **Name** of `Extension` and **Value** of `soc2-posture` to narrow down the results. There are some resources in this region with that tag that will deliberately generate SOC2 Security findings. 
5. Provide a link to the GitHub repo that includes the Terraform to also be considered in remediation: `https://github.com/duplo-darren/duplo-aws-workshop-sep2-2026`
6. Select the minimum severity of findings to include.
7. Start the assessment and allow it to run to completion _(click **Track Provisioning Status** to follow the agent's progress)_.

### Reading the Results

The assessment produces a findings table. Key things to note:

- **Source indicator** — a pill/badge appears on rows where the resource is managed by Terraform, indicating that any fix should also be delivered via Terraform to keep infrastructure-as-code consistent. These rows are also shaded for ease of identification. 
- **Resolve button** — appears at the end of each row and triggers the remediation workflow for that finding.

---

## Step 7: Remediate Findings

This extension supports two remediation paths depending on how the resource is managed.

---

### Option A: CLI Remediation (Non-Terraform Resources)

**Example finding:** *Account-level S3 Block Public Access is not fully enabled*

1. Locate the finding in the table and click **Resolve**.
2. Select the **AWS Instance Role** scope.
3. Click **Resolve** — DuploCloud will surface the AWS CLI command needed to apply the fix, along with the affected account details.
4. Review the command. If you want to apply it directly, click **Execute Fix** — DuploCloud will use the AWS scope to run the command against the account.

> CLI remediations are surfaced for human review first. You decide whether and when to apply them.

---

### Option B: Terraform Pull Request (Terraform-Managed Resources)

**Example finding:** *Security group allows RDP port 3389 open to the world*

This flow requires a **GitHub scope**. If you haven't added one yet, do so first. You can use your own personal GitHub account which will fork the repo and result in a Pull Request. 

#### Add a GitHub Source Control Provider

1. Go to **DuploCloud Admin → Providers → Source Control**.
2. Click **Create New Provider**.
3. Set the name to `GitHub`, leave the type as **GitHub.com** (self-hosted GitHub Enterprise is also supported if needed).
4. Provide a **personal access token** with access to your GitHub account.
5. Give the scope a name (e.g., `GitHub Scope`) and click **Create**.
6. When prompted, click **Attach** to add it to your DevKit workspace.

#### Raise a Remediation Pull Request

1. Return to the SOC 2 Posture findings.
2. Find the **RDP 3389 open to world** finding and click **Resolve**.
3. Select your **GitHub Scope** and **AWS Instance Role**.
4. Click **Resolve** — DuploCloud opens a child ticket to generate the Terraform fix.
5. Click **Track Provision Status** to follow progress.
6. When prompted for a value (e.g., allowed IP/CIDR range), click **Provide Value**, enter the IP with subnet mask (e.g., `8.8.8.8/32`), and click **Submit**.
7. DuploCloud reruns the remediation script with the value you provided and generates a pull request.
8. Click **View Pull Request** to inspect the diff in GitHub — confirm the change looks correct, then merge if satisfied.

> Delivering the fix via pull request keeps your infrastructure-as-code source of truth consistent. New environments provisioned from the same Terraform repository will also inherit the fix.

---

### Reviewing All Remediations

Navigate to the **Remediations** tab (top-right of the SOC 2 Posture view) to see a summary of all remediation tickets, their current status, and any items awaiting your input.

---

## Step 8: Iterate on the Extension

A core strength of DevKit is that you can iterate on an extension without starting from scratch. The coding agent has visibility into the extension source code and can make targeted changes, rebuild, and redeploy — all from a single prompt.

### Add the Availability Trust Services Criteria

1. Return to Code Server — Claude should still be at the prompt with Auto Mode active.
2. Enter the following:

   ```
   Enable the Availability trust services criteria for the soc2-posture extension.
   ```

3. Claude will analyze the existing extension code, implement the required additions, rebuild the extension, and push it to DuploCloud.
4. Once complete, return to DuploCloud and open **SOC 2 Posture**.
5. Click **New Assessment** — **Availability** will now appear as an available criteria option.

You can continue iterating: add criteria one at a time to validate each addition, or enable all remaining criteria in a single pass.

---

## What's Next?

Your workshop environment stays available after the session. Some things to try on your own:

- Run a full SOC 2 assessment with all trust service criteria enabled
- Explore the Kubernetes scope and run queries against cluster resources
- Build a net-new extension from scratch using the `duplo-extension-dev` skill wizard
- Experiment with additional remediation workflows and pull request flows
- Explore the DuploCloud admin interface and workspace/scope configuration

---

## Support

If you run into any issues during the workshop, flag a facilitator or reach out via the workshop support channel.

---

*Workshop content provided by DuploCloud.*
