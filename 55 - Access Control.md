## Introduction to Access Control

- Access control determines which system resources or services can be accessed by which parties under which circumstances
- Two key steps:
    1. Determine if request fits security policy
    2. Perform operation if allowed, deny if not
- Access control decisions are typically binary (yes/no)

## Important Aspects of Access Control

- Involves subjects (entities wanting access), objects (things being accessed), and access modes
- Authorization process determines if a subject can perform a particular access on an object
- Reference monitor implements access control algorithm
- Need to balance security and performance
- Virtualization can help with some special cases (e.g. virtual memory)

## Access Control Lists (ACLs)

- Associates list of allowed accesses with each object
- When access requested, system checks if subject is on object's ACL
- Advantages:
    - Easy to see who can access an object
    - Easy to revoke access by removing from ACL
- Disadvantages:
    - Can be expensive to store and search large ACLs
    - Hard to determine all objects a subject can access

## Capabilities

- Subject possesses unforgeable tokens (capabilities) that grant access rights
- To access object, subject presents relevant capability
- Advantages:
    - Easy to determine what a subject can access
    - Can support fine-grained privilege
- Disadvantages:
    - Hard to revoke access
    - Hard to determine all subjects that can access an object

## Practical Considerations

- Most systems use ACLs for user-visible access control
- Capabilities often used internally by OS
- Default permissions important since users rarely change them
- Role-based access control (RBAC) useful for managing permissions in large organizations

## Mandatory vs Discretionary Access Control

- Discretionary: Resource owner controls access
- Mandatory: Some access controlled by system policy, can't be overridden by owner

## Summary

- Access control critical for implementing security policies
- ACLs and capabilities are two main approaches, each with pros and cons
- Practical systems often combine techniques
- Well-designed policies as important as mechanisms
