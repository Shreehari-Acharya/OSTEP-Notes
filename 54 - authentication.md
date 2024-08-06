## Introduction

- Authentication is crucial for implementing security policies in operating systems
- The OS needs to know the identity of processes to make proper security decisions
- Authentication starts with a process at boot time that authenticates users

### Key Concepts:
- Principal: The party requesting access (e.g. human user, group, software system)
- Agent: The process or entity acting on behalf of a principal
- Object: The resource being accessed
- Credential: Data that keeps track of access decisions

## Types of Authentication

### Authentication by What You Know
- Most commonly implemented using passwords
- Passwords should be:
  - Long 
  - Composed of many character types
  - Not based on common words/patterns
- Password files should store hashed and salted passwords, not plaintext
- Dictionary attacks are a risk for weak passwords

### Authentication by What You Have  
- Examples: ID cards, security tokens, smart cards
- Can be combined with passwords for multi-factor authentication
- Challenges:
  - Loss/theft of authentication device
  - Need for special hardware readers in some cases

### Authentication by What You Are (Biometrics)
- Uses unique physical or behavioral characteristics 
- Examples: fingerprints, facial recognition, typing patterns
- Challenges:
  - Requires special hardware in many cases
  - False positive and false negative errors
  - Crossover error rate as a metric of accuracy
  - Risk of remote spoofing attacks

## Authenticating Non-Human Entities

- Used for processes not associated with human users (e.g. web servers, embedded devices)
- Approaches:
  - Create special user accounts (e.g. "webserver")
  - Use passwords for these accounts
  - Allow privileged users to assign identities (e.g. sudo command)
  - Temporary identity changes

## Group Authentication

- Identifies groups of users with shared privileges
- Allows flexible management of access rights
- Supported by most modern operating systems
- Provides an additional path for access, with benefits and risks

## Summary

- Authentication is crucial for applying security policies to processes
- Common methods: passwords, tokens/cards, biometrics 
- Multi-factor authentication increases security
- Special considerations needed for non-human entities and groups
- Each method has strengths and weaknesses to consider
