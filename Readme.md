# SafeGuard - Family Safety & Child Monitoring App

SafeGuard is a comprehensive family safety and child monitoring application designed to help parents ensure the well-being of their children in the digital age. With features like location tracking, screen time management, and content filtering, SafeGuard provides peace of mind for parents while promoting healthy digital habits for kids.

## About the Developer

- **Name**: Mai Atef Elkheshen
- **Education**: Final year student at computer and information science menofia University.
- **Project**: Graduation Project - SafeGuard Family Safety App

## My Roles in This Project

### 1. Leadership & Ownership
- Proposed and owned the original project idea, aligning it with real-world family safety needs.  
- Acted as **Team Lead**, organizing tasks, ensuring timely progress, and coordinating collaboration between frontend, backend, and design teams.  
- Facilitated communication between team members, resolving conflicts, and maintaining a positive team dynamic.

### 2. UI/UX Design
- Designed a **child-friendly and parent-focused interface** that balances usability and safety.  
- Delivered detailed prototypes for implementation in **Flutter**, ensuring consistency across platforms.  
- Focused on accessibility, ensuring the app is easy to use for both tech-savvy and non-tech-savvy parents.

### 3. Documentation
- Authored project documentation including **user stories, use case diagrams, sequence diagrams, and technical specifications** to guide development.  


### 4. User Research & Feedback
- Conducted **competitive analysis** of similar apps (e.g., FindMyKids, Life360).  
- Collected and analyzed **user feedback** to iteratively improve usability and feature alignment.  
- [Example reference](https://play.google.com/store/apps/details?id=org.findmykids.app&hl=en-US)  
---
## all implementation is in progress 

### 5. Backend Integration (In Progress)
- Collaborating with backend developers to integrate APIs between the **Flutter frontend** and **Spring Boot REST API backend**.  
- Ensured smooth data flow, authentication, and secure communication with the **PostgreSQL database**.  



### 6. Testing & Debugging (In Progress)
- Leading **quality assurance efforts**, identifying bugs, and validating app performance.  
- Coordinating testing between multiple devices (parent and child apps).  


### 7. Team Collaboration (In Progress)
- Facilitated **cross-functional teamwork**, ensuring seamless integration of features.  
- Supported backend engineers while guiding the implementation of frontend designs.  
- Promoted a culture of continuous improvement and learning within the team.

## Project Overview

SafeGuard consists of two main applications:
- **Parent App**: A monitoring dashboard for parents to track children's safety, manage geofences, receive AI-powered alerts, and coordinate family safety.
- **Child App**: A child-friendly interface that allows children to check trusted faces, view schedules, communicate with parents, and access safety features in an engaging, age-appropriate manner.

## Key Features

### Parent App Features
- **Authentication & Onboarding**: Secure registration/login with email/phone, social login, and biometric support
- **Real-Time Dashboard**: Comprehensive family safety status with location tracking and battery monitoring
- **Child Management**: Add/manage children with medical info, emergency contacts, and device pairing
- **Location Tracking**: Interactive maps with real-time updates and location history
- **Geofencing**: Create safe zones with customizable alerts for entry/exit events
- **AI-Powered Monitoring**: Audio detection, keyword detection, behavioral anomaly detection, face recognition (with make sure to comply with privacy laws and obtain necessary consents and keep models work offline to save privacy)

- **Smart Notifications**: Prioritized alerts with customizable preferences and quiet hours
- **Family Management**: Invite other parents, share child monitoring, and coordinate schedules
- **Emergency Features**: SOS functionality, emergency contacts, and rapid response coordination
- **Reports & Analytics**: Safety reports, location heat maps, and alert analytics

### Child App Features
- **Child-Friendly Interface**: Colorful, engaging design with large buttons and simple navigation
- **Trusted Face Recognition**: Easy scanning to identify trusted people with positive feedback
- **Schedule Management**: Fun display of daily activities with reminders and completion rewards
- **Safety Features**: SOS emergency button, safe zone alerts, and stranger warnings
- **Gamification**: Achievement system with badges, stars, and rewards for safe behavior
- **Personalization**: Customizable themes, avatars, and sound preferences

## Technical Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Parent App    │    │   Child App     │    │   Web Admin     │
│   (Flutter)     │    │   (Flutter)     │    │   (Optional)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Spring Boot   │
                    │   REST API      │
                    │   (Backend)     │
                    └─────────────────┘
                             │
                    ┌─────────────────┐
                    │   PostgreSQL    │
                    │   (Database)    │
                    └─────────────────┘
```
