# Apache Kafka Course - Hands-On Exercises

Welcome to the hands-on exercises section! These exercises are designed to reinforce the concepts covered in each module and provide practical experience with Apache Kafka.

## � Table of Contents

- [📚 Exercise Structure](#-exercise-structure)
- [🎯 Learning Approach](#-learning-approach)
- [📋 Exercise Modules](#-exercise-modules)
  - [Module 1: Introduction to Apache Kafka](#module-1-introduction-to-apache-kafka)
  - [Module 2: Kafka Architecture and Core Concepts](#module-2-kafka-architecture-and-core-concepts)
  - [Module 3: Setting Up Kafka](#module-3-setting-up-kafka)
  - [Module 4: Kafka Producers](#module-4-kafka-producers)
  - [Module 5: Kafka Consumers](#module-5-kafka-consumers)
  - [Module 6: Topics, Partitions, and Data Management](#module-6-topics-partitions-and-data-management)
  - [Module 7: Kafka Connect and Kafka Streams](#module-7-kafka-connect-and-kafka-streams)
  - [Module 8: Advanced Topics and Best Practices](#module-8-advanced-topics-and-best-practices)
  - [Module 9: Practical Application Project](#module-9-practical-application-project)
- [🛠️ Prerequisites](#️-prerequisites)
  - [Required Software](#required-software)
  - [Optional Software](#optional-software)
  - [System Requirements](#system-requirements)
- [📖 How to Use These Exercises](#-how-to-use-these-exercises)
  - [For Beginners](#for-beginners)
  - [For Intermediate Users](#for-intermediate-users)
  - [For Advanced Users](#for-advanced-users)
- [🎓 Learning Tips](#-learning-tips)
- [🤝 Getting Help](#-getting-help)
- [📊 Progress Tracking](#-progress-tracking)
- [📁 Exercise Files Structure](#-exercise-files-structure)
- [🚀 Next Steps](#-next-steps)

## �📚 Exercise Structure

Each exercise module includes:
- **Guided exercises** with step-by-step instructions
- **Challenge exercises** to test your understanding
- **Solutions and discussion** for self-assessment
- **Time estimates** for planning your learning

## 🎯 Learning Approach

**Recommended workflow:**
1. Complete the corresponding theory module first
2. Work through exercises sequentially
3. Attempt challenges before viewing solutions
4. Build hands-on experience incrementally
5. Create your own variations and experiments

## 📋 Exercise Modules

### [Module 1: Introduction to Apache Kafka](module-01-exercises.md)
**Time:** 2-3 hours | **Difficulty:** Beginner

**Topics covered:**
- Understanding event streaming concepts
- Kafka use cases analysis
- Comparing Kafka with message brokers
- Designing event-driven architectures
- Exploring Kafka ecosystem

**Key exercises:**
- Event streaming vs traditional messaging comparison
- Real-world use case analysis
- Event-driven architecture design
- ROI calculator challenge

---

### [Module 2: Kafka Architecture and Core Concepts](module-02-exercises.md)
**Time:** 3-4 hours | **Difficulty:** Beginner-Intermediate

**Topics covered:**
- Kafka cluster architecture
- Partition assignment strategies
- Offset management
- Replication and ISR
- Log segments and retention
- Controller operations

**Key exercises:**
- Cluster architecture diagrams
- Partition assignment simulation
- Offset management strategies
- Replication scenarios
- Multi-datacenter design challenge

---

### [Module 3: Setting Up Kafka](module-03-exercises.md)
**Time:** 4-5 hours | **Difficulty:** Intermediate

**Topics covered:**
- Local installation
- Docker Compose setup
- CLI tools mastery
- Kafka UI setup
- Configuration tuning
- Performance benchmarking

**Key exercises:**
- 3-broker cluster setup
- Topic management via CLI
- Producer and consumer CLI practice
- Performance benchmarking
- HA cluster challenge

---

### [Module 4: Kafka Producers](module-04-exercises.md)
**Time:** 6-7 hours | **Difficulty:** Intermediate

**Topics covered:**
- Basic producer implementation
- Synchronous vs asynchronous sending
- Custom serializers
- Custom partitioners
- Producer configuration for reliability
- Error handling and retries
- Transaction support

**Key exercises:**
- Implement producers in TypeScript/Node.js
- Create custom serializer and partitioner
- Configure for throughput vs reliability
- Build resilient producer with DLQ
- Production-ready producer service challenge

---

### [Module 5: Kafka Consumers](module-05-exercises.md)
**Time:** 6-7 hours | **Difficulty:** Intermediate-Advanced

**Topics covered:**
- Basic consumer implementation
- Consumer groups and load balancing
- Offset management strategies
- Handling rebalancing
- Consumer lag monitoring and reduction
- Error handling and DLQ
- Exactly-once semantics

**Key exercises:**
- Consumer group behavior simulation
- Implement 5 offset commit strategies
- Rebalance listener implementation
- Consumer lag monitoring and optimization
- Real-time analytics consumer challenge

---

### [Module 6: Topics, Partitions, and Data Management](module-06-exercises.md)
**Time:** 5-6 hours | **Difficulty:** Intermediate

**Topics covered:**
- Topic design and planning
- Partition count optimization
- Retention configuration
- Log compaction
- Schema management
- Storage capacity planning
- Topic administration

**Key exercises:**
- Design topics for use cases
- Calculate optimal partition count
- Configure retention policies
- Implement compacted topic workflow
- Schema evolution with Schema Registry
- Multi-tiered storage challenge

---

### [Module 7: Kafka Connect and Kafka Streams](module-07-exercises.md)
**Time:** 6-7 hours | **Difficulty:** Advanced

**Topics covered:**
- JDBC source connector
- Elasticsearch sink connector
- S3 sink with partitioning
- Custom SMTs
- Kafka Streams word count
- Real-time analytics with Streams
- State stores and interactive queries

**Key exercises:**
- Set up multiple connectors
- Create custom transformation
- Build word count application
- Real-time clickstream analytics
- Interactive query service
- Complex event processing challenge

---

### [Module 8: Advanced Topics and Best Practices](module-08-exercises.md)
**Time:** 7-8 hours | **Difficulty:** Advanced

**Topics covered:**
- SSL/TLS security setup
- SASL authentication
- ACL management
- Performance tuning (OS, broker, clients)
- Monitoring and alerting
- Disaster recovery
- Capacity planning
- Production readiness

**Key exercises:**
- Implement complete security setup
- Tune for maximum performance
- Set up comprehensive monitoring
- Test disaster recovery scenarios
- Complete capacity planning
- Production readiness checklist challenge

---

### [Module 9: Practical Application Project](module-09-exercises.md)
**Time:** 12-16 hours | **Difficulty:** Advanced

**Capstone project: Real-Time E-Commerce Analytics Platform**

**Build a complete system including:**
- Event producer services (REST API, Order Service, Inventory)
- Stream processing (Enrichment, Analytics, Fraud Detection)
- Consumer services (Notifications, Recommendations)
- Data integration (Elasticsearch, PostgreSQL, S3)
- Monitoring and operations (Prometheus, Grafana)
- Testing and deployment

**Project phases:**
1. Architecture design
2. Infrastructure setup
3. Producer services implementation
4. Stream processing pipelines
5. Consumer services
6. Data integration
7. Monitoring
8. Testing and validation
9. Deployment and optimization

---

## 🛠️ Prerequisites

### Required Software
- **Node.js:** Version 16 or higher
- **npm/yarn:** Package manager
- **TypeScript:** ^4.9 or higher
- **Docker:** Latest version with Docker Compose
- **Git:** For version control
- **IDE:** IntelliJ IDEA, VS Code, or Eclipse
- **Terminal:** Bash, Zsh, or PowerShell

### Optional Software
- **Python:** 3.8+ (for Python examples)
- **Node.js:** 16+ (for Node.js examples)
- **kubectl:** For Kubernetes exercises
- **npm/yarn:** For TypeScript projects

### System Requirements
- **RAM:** Minimum 8 GB, recommended 16 GB
- **Disk:** Minimum 20 GB free space
- **CPU:** Multi-core processor (4+ cores recommended)
- **OS:** Linux, macOS, or Windows 10/11

## 📖 How to Use These Exercises

### For Beginners
1. Start with Module 1 and progress sequentially
2. Don't skip exercises - hands-on practice is essential
3. Use provided solutions as learning aids
4. Join Kafka community forums for help
5. Build confidence with each module

### For Intermediate Users
1. Review modules where you need reinforcement
2. Focus on challenge exercises
3. Experiment with variations
4. Compare your solutions with provided ones
5. Build portfolio projects

### For Advanced Users
1. Jump to advanced modules (6-9)
2. Complete challenge exercises first
3. Extend exercises with additional features
4. Contribute solutions to community
5. Mentor others on their journey

## 🎓 Learning Tips

**Maximize your learning:**
- ✅ **Hands-on practice:** Type code yourself, don't copy-paste
- ✅ **Break things:** Learn by causing and fixing errors
- ✅ **Read logs:** Understanding errors makes you better
- ✅ **Measure everything:** Use monitoring to understand behavior
- ✅ **Ask why:** Don't just follow steps, understand reasons
- ✅ **Document:** Keep notes on what you learn
- ✅ **Teach others:** Best way to solidify understanding

**Common pitfalls to avoid:**
- ❌ Skipping theory modules
- ❌ Rushing through exercises
- ❌ Not testing failure scenarios
- ❌ Ignoring performance implications
- ❌ Skipping cleanup (topics, consumer groups)
- ❌ Not reading error messages carefully

## 🤝 Getting Help

**Resources:**
- **Kafka Documentation:** https://kafka.apache.org/documentation/
- **Confluent Docs:** https://docs.confluent.io/
- **Stack Overflow:** Tag `apache-kafka`
- **Kafka Users Mailing List:** users@kafka.apache.org
- **Community Slack:** https://kafka-users.slack.com

**Troubleshooting:**
1. Check broker logs: `docker logs kafka-1`
2. Verify topic configuration: `kafka-topics.sh --describe`
3. Check consumer group status: `kafka-consumer-groups.sh --describe`
4. Review error messages carefully
5. Search issues on GitHub and Stack Overflow

## 📊 Progress Tracking

Track your learning progress:

```
Module 1 (Introduction)              [ ] Started  [ ] Completed
Module 2 (Architecture)              [ ] Started  [ ] Completed
Module 3 (Setup)                     [ ] Started  [ ] Completed
Module 4 (Producers)                 [ ] Started  [ ] Completed
Module 5 (Consumers)                 [ ] Started  [ ] Completed
Module 6 (Data Management)           [ ] Started  [ ] Completed
Module 7 (Connect & Streams)         [ ] Started  [ ] Completed
Module 8 (Advanced Topics)           [ ] Started  [ ] Completed
Module 9 (Practical Application)     [ ] Started  [ ] Completed
```

**Estimated total time:** 50-60 hours of hands-on practice

## 📁 Exercise Files Structure

```
exercise/
├── README.md (this file)
├── module-01-exercises.md
├── module-02-exercises.md
├── module-03-exercises.md
├── module-04-exercises.md
├── module-05-exercises.md
├── module-06-exercises.md
├── module-07-exercises.md
├── module-08-exercises.md
└── module-09-exercises.md
```

## 🚀 Next Steps

1. **Start with Module 1** if you're new to Kafka
2. **Review prerequisites** and install required software
3. **Set up your development environment**
4. **Join the community** for support
5. **Begin your Kafka journey!**

---

**Ready to get started?**

**[Begin with Module 1 Exercises →](module-01-exercises.md)**

**Or jump to a specific module:**
- [Module 2: Architecture](module-02-exercises.md)
- [Module 3: Setup](module-03-exercises.md)
- [Module 4: Producers](module-04-exercises.md)
- [Module 5: Consumers](module-05-exercises.md)
- [Module 6: Data Management](module-06-exercises.md)
- [Module 7: Connect & Streams](module-07-exercises.md)
- [Module 8: Advanced Topics](module-08-exercises.md)
- [Module 9: Practical Application](module-09-exercises.md)

---

**[📖 Back to Course Home](../README.md)**
