# Deployment Documentation

Complete deployment guides for the MHFA Learning Navigator chatbot.

---

## 🚀 Quick Start (Recommended)

**New to the project?** Start here:

### ⚡ CloudShell Deployment (3 Commands - 25 minutes)
The simplest way to deploy. Runs entirely in AWS CloudShell - no local setup required.

**📄 [CLOUDSHELL_DEPLOY.md](../../CLOUDSHELL_DEPLOY.md)** - Quick reference guide (in project root)

**📖 [CLOUDSHELL_DEPLOYMENT_EXPLAINED.md](CLOUDSHELL_DEPLOYMENT_EXPLAINED.md)** - Detailed explanation
- Line-by-line command breakdown
- What happens behind the scenes
- 50+ AWS resources explained
- Cost breakdown
- Security details
- Troubleshooting guide

---

## 📚 All Deployment Methods

Choose the method that works best for you:

### 1. CloudShell Deploy (Simplest) ⚡
**Time:** 25-30 minutes | **Difficulty:** Easy | **Setup:** None

Perfect for first-time deployments or quick testing.

- **[Quick Guide](../../CLOUDSHELL_DEPLOY.md)** - 3 commands, copy-paste ready
- **[Detailed Explanation](CLOUDSHELL_DEPLOYMENT_EXPLAINED.md)** - Step-by-step breakdown

**Best for:**
- First-time users
- Quick deployments
- No local environment setup
- Testing before production

---

### 2. 4-Command Local Deploy 🚀
**Time:** 30 minutes | **Difficulty:** Easy | **Setup:** AWS CLI, Git

Fast automated deployment from your local machine.

- **[4-COMMAND-DEPLOY.md](4-COMMAND-DEPLOY.md)** - Automated CodeBuild deployment

**Best for:**
- Developers with AWS CLI configured
- CI/CD pipeline integration
- Repeated deployments

---

### 3. 5-Step Manual Deploy 📝
**Time:** 2 hours | **Difficulty:** Moderate | **Setup:** AWS CLI, CDK, Node.js

Step-by-step manual deployment with full control.

- **[QUICK_DEPLOY.md](QUICK_DEPLOY.md)** - 5-step walkthrough

**Best for:**
- Learning the infrastructure
- Custom configurations
- Understanding each component

---

### 4. Comprehensive Deployment 📚
**Time:** 2-3 hours | **Difficulty:** Advanced | **Setup:** Full development environment

Complete guide with advanced options and troubleshooting.

- **[DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)** - Comprehensive documentation

**Best for:**
- Production deployments
- Advanced configurations
- Troubleshooting complex issues
- Enterprise requirements

---

### 5. Deployment Checklist ✅
**Time:** Varies | **Difficulty:** Varies | **Setup:** Depends on method

Checkbox-style reference for any deployment method.

- **[DEPLOYMENT_CHECKLIST.md](DEPLOYMENT_CHECKLIST.md)** - Step-by-step checklist

**Best for:**
- Tracking progress
- Team deployments
- Audit trails
- Quality assurance

---

## 💰 Cost Information

Understand deployment costs before starting:

### Simple Cost Guide
**[COST_ESTIMATION_SIMPLE.md](COST_ESTIMATION_SIMPLE.md)** - Quick overview
- Pricing tiers (Low/Medium/High/Enterprise)
- Monthly cost ranges
- Usage-based estimates

### Detailed Cost Analysis
**[COST_ESTIMATION.md](COST_ESTIMATION.md)** - Comprehensive breakdown
- Service-by-service costs
- Optimization strategies
- Cost calculators
- Monitoring setup

**Quick Summary:**
- **Low usage** (1K conversations/month): $25-40
- **Medium usage** (10K conversations/month): $138-208
- **High usage** (50K conversations/month): $648-948

---

## 📋 Customer Resources

Share with clients or stakeholders:

### Customer Deployment Brief
**[CUSTOMER_DEPLOYMENT_BRIEF.md](CUSTOMER_DEPLOYMENT_BRIEF.md)**
- Executive overview
- Deployment options comparison
- Timeline and cost estimates
- Prerequisites checklist

---

## 📊 Deployment Comparison

| Method | Time | Commands | Difficulty | Best For |
|--------|------|----------|------------|----------|
| **CloudShell** ⚡ | 25-30 min | 3 | Easiest | First-time users |
| **4-Command Local** 🚀 | 30 min | 4 | Easy | Developers |
| **5-Step Manual** 📝 | 2 hours | 15+ | Moderate | Custom setups |
| **Comprehensive** 📚 | 2-3 hours | 20+ | Advanced | Full control |
| **Checklist** ✅ | Varies | Varies | Reference | Any method |

---

## 🎯 Deployment Prerequisites

All methods require:

### 1. AWS Account Setup
- AWS account with admin access
- AWS CLI configured (for local deployments)
- Region: us-west-2 (Oregon) recommended

### 2. Enable Bedrock Models
Go to https://console.aws.amazon.com/bedrock/
- ✅ Anthropic Claude 4 Sonnet
- ✅ Amazon Titan Embeddings Text v2

### 3. Verify Admin Email
Go to https://console.aws.amazon.com/ses/
- Verify your admin email address
- Click verification link in email

### 4. Prepare Documents
- Upload PDFs, Word docs, or text files to S3 bucket
- Organize in `pdfs/` folder
- Note the bucket name

**Detailed instructions in each deployment guide.**

---

## 🆘 Need Help?

### Troubleshooting
Each deployment guide includes:
- Common errors and solutions
- Debug commands
- Log analysis
- Support resources

### Additional Documentation
- **[Main README](../../README.md)** - Project overview
- **[Technical Documentation](../TECHNICAL_DOCUMENTATION_SIMPLE.md)** - Architecture
- **[Admin Features](../features/ADMIN_FEATURES.md)** - Admin portal guide

---

## 📝 Deployment Workflow

```
1. Choose Deployment Method
   ↓
2. Review Prerequisites
   ↓
3. Prepare Your Environment
   ↓
4. Follow Deployment Guide
   ↓
5. Verify Deployment
   ↓
6. Sync Knowledge Base
   ↓
7. Create Admin Users
   ↓
8. Test Application
   ↓
9. Share with Users
```

---

## 🎉 After Deployment

Once deployed, you'll have:

**✅ Live Chatbot:**
- Public URL: `https://main.xxxxx.amplifyapp.com`
- Real-time AI chat interface
- Multilingual support (English/Spanish)
- Role-based recommendations
- No login required for users

**✅ Admin Dashboard:**
- URL: `https://main.xxxxx.amplifyapp.com/admin`
- Document management
- Analytics and sentiment analysis
- Conversation logs
- Escalated query management
- Cognito authentication required

**✅ AWS Infrastructure:**
- 50+ AWS resources deployed
- Fully managed serverless architecture
- Auto-scaling and high availability
- CloudWatch monitoring
- Cost-optimized configuration

---

## 📞 Support

**Questions about deployment?**
1. Check the specific deployment guide troubleshooting section
2. Review CloudWatch logs for errors
3. Consult the comprehensive deployment guide
4. Check AWS Service Health Dashboard

**Repository:** https://github.com/ASUCICREPO/NCMW-Learning-Navigator-chatbot

---

**Last Updated:** January 29, 2026
**Documentation Version:** 2.0
