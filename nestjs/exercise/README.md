# NestJS Course Exercises

Welcome to the hands-on exercises for the NestJS Course! This directory contains comprehensive, practical exercises for each module of the course.

## 📚 Exercise Structure

Each module has dedicated exercises that build upon concepts learned in the corresponding module content.

### Module 1: Introduction to NestJS
**[Module 1 Exercises](module-01-exercises.md)**

Learn NestJS fundamentals through research and conceptual exercises:
- Compare NestJS with other frameworks
- Study SOLID principles with code examples
- Design application architecture
- Understand OOP vs FP approaches
- Compare Express vs Fastify platforms

**Time Commitment:** 6-8 hours  
**Difficulty:** Beginner  
**Type:** Research & Conceptual

---

### Module 2: Getting Started
**[Module 2 Exercises](module-02-exercises.md)**

Set up your development environment and create your first NestJS application:
- Create your first NestJS app (3 methods)
- Explore and document project structure
- Configure development workflow (ESLint, Prettier, etc.)
- Set up multi-environment configuration
- Create custom project templates

**Time Commitment:** 4-6 hours  
**Difficulty:** Beginner  
**Type:** Setup & Configuration

---

### Module 3: Controllers
**[Module 3 Exercises](module-03-exercises.md)**

Master HTTP request handling and routing:
- Build complete CRUD controller
- Implement query parameters and filtering
- Create nested routes (users → posts)
- Work with request/response customization
- Create and use DTOs

**Time Commitment:** 5-7 hours  
**Difficulty:** Beginner to Intermediate  
**Type:** Hands-on Coding

---

### Module 4: Providers and Dependency Injection
**[Module 4 Exercises](module-04-exercises.md)**

Learn dependency injection and service patterns:
- Create full-featured UserService
- Implement Repository pattern
- Build custom providers (factory, async, etc.)
- Design multi-layer architecture
- Create E-commerce backend challenge

**Time Commitment:** 6-8 hours  
**Difficulty:** Intermediate  
**Type:** Architecture & Design

---

### Module 5: Modules
**[Module 5 Exercises](module-05-exercises.md)**

Organize applications with modular architecture:
- Create feature modules (Users, Auth, Products, Orders)
- Build SharedModule with common services
- Implement dynamic modules with forRoot/forFeature
- Refactor monolithic applications
- Design plugin system

**Time Commitment:** 5-7 hours  
**Difficulty:** Intermediate  
**Type:** Architecture & Organization

---

### Module 6: Core Fundamentals
**[Module 6 Exercises](module-06-exercises.md)**

Master exception handling, testing, and best practices:
- Create custom exception filters
- Write comprehensive unit tests
- Build E2E test suites
- Set up configuration with validation
- Apply SOLID principles in practice

**Time Commitment:** 6-8 hours  
**Difficulty:** Intermediate to Advanced  
**Type:** Testing & Best Practices

---

### Module 7: Additional Fundamentals
**[Module 7 Exercises](module-07-exercises.md)**

Implement middleware, pipes, guards, and interceptors:
- Create logging middleware with request duration
- Build validation pipes with class-validator
- Implement JWT authentication guards
- Create response transformation interceptors
- Design custom decorators

**Time Commitment:** 5-7 hours  
**Difficulty:** Intermediate to Advanced  
**Type:** Request Lifecycle Management

---

### Module 8: Practical Application
**[Module 8 Exercises](module-08-exercises.md)**

Build a complete production-ready Task Management API:
- Phase 1: Project setup and structure
- Phase 2: User management with authentication
- Phase 3: JWT-based auth implementation
- Phase 4: Task CRUD with filtering
- Phase 5: Global features (interceptors, filters)
- Phase 6: Comprehensive testing

**Time Commitment:** 10-15 hours  
**Difficulty:** Advanced  
**Type:** Full Project Implementation

---

## 🎯 Learning Approach

### Recommended Path

1. **Read Module Content First**
   - Review the corresponding module content file
   - Understand concepts before coding
   - Take notes on key points

2. **Work Through Exercises Sequentially**
   - Don't skip exercises
   - Build on previous work
   - Complete all tasks before moving on

3. **Follow the Time Estimates**
   - Allocate proper time for each exercise
   - Don't rush - understanding is key
   - Take breaks between sessions

4. **Test Everything**
   - Run your code frequently
   - Fix errors immediately
   - Verify outputs match expectations

5. **Document Your Work**
   - Add comments to code
   - Create README files
   - Note challenges and solutions

### Exercise Difficulty Levels

- 🟢 **Beginner:** Fundamental concepts, guided instructions
- 🟡 **Intermediate:** More independence required, complex scenarios
- 🔴 **Advanced:** Architectural decisions, best practices, optimization

---

## 💡 Tips for Success

### Before Starting
- ✅ Ensure Node.js >= 20.0.0 is installed
- ✅ Have VS Code or preferred editor ready
- ✅ Install NestJS CLI globally: `npm i -g @nestjs/cli`
- ✅ Familiarize yourself with TypeScript basics
- ✅ Review corresponding module content

### While Working
- 💾 Commit code frequently to git
- 🧪 Test each feature as you build
- 📝 Document code and decisions
- 🔍 Read error messages carefully
- 🔄 Refactor code as you learn better patterns
- ❓ Research when stuck (docs, Stack Overflow)

### After Completing
- ✔️ Review all checklist items
- 📊 Analyze code quality
- 🎨 Refactor for improvements
- 📚 Compare with official examples
- 🤔 Reflect on what you learned

---

## 📁 Project Organization

### Recommended Workspace Structure

```
nestjs-exercises/
├── module-01-concepts/
│   ├── framework-comparison.md
│   ├── solid-examples/
│   └── architecture-design.md
├── module-02-setup/
│   ├── my-first-nest-app/
│   ├── nest-cli-project/
│   └── nest-git-clone/
├── module-03-controllers/
│   └── users-api/
├── module-04-providers/
│   ├── users-service-app/
│   └── products-architecture/
├── module-05-modules/
│   └── modular-app/
├── module-06-fundamentals/
│   └── exception-handling-app/
├── module-07-lifecycle/
│   └── middleware-app/
└── module-08-project/
    └── task-management-api/
```

### Managing Multiple Projects

**Option 1: Separate Folders**
```bash
mkdir nestjs-exercises
cd nestjs-exercises
# Create separate project for each exercise
```

**Option 2: Monorepo with Nx**
```bash
npx create-nx-workspace@latest nestjs-exercises
```

**Option 3: Single Project, Multiple Branches**
```bash
git checkout -b module-01-exercises
# Complete exercises
git checkout -b module-02-exercises
# Continue...
```

---

## 🛠️ Common Tools & Resources

### Development Tools
- **NestJS CLI:** Project generation and scaffolding
- **Postman/Insomnia:** API testing
- **cURL:** Command-line API testing
- **REST Client (VS Code):** In-editor API testing

### Testing Tools
- **Jest:** Unit and integration testing
- **Supertest:** E2E API testing
- **Artillery/K6:** Load testing (bonus)

### Debugging Tools
- **VS Code Debugger:** Breakpoints and inspection
- **Chrome DevTools:** Node.js debugging
- **Nest CLI Logs:** Application logging

---

## 📖 Reference Materials

### Official Documentation
- [NestJS Documentation](https://docs.nestjs.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Node.js Documentation](https://nodejs.org/docs/)

### Community Resources
- [NestJS Discord](https://discord.gg/nestjs)
- [Stack Overflow - NestJS Tag](https://stackoverflow.com/questions/tagged/nestjs)
- [GitHub NestJS Examples](https://github.com/nestjs/nest/tree/master/sample)

### Additional Learning
- [NestJS Courses on Udemy](https://www.udemy.com/topic/nestjs/)
- [Official NestJS Courses](https://courses.nestjs.com/)
- [YouTube Tutorials](https://www.youtube.com/results?search_query=nestjs)

---

## ✅ Progress Tracking

Use this checklist to track your progress:

### Module 1: Introduction
- [ ] Framework comparison completed
- [ ] SOLID principles examples created
- [ ] Architecture design document written
- [ ] OOP vs FP comparison done
- [ ] Platform comparison finished

### Module 2: Getting Started
- [ ] First app created and running
- [ ] Project structure documented
- [ ] All installation methods tested
- [ ] Development workflow configured
- [ ] Multi-environment setup complete

### Module 3: Controllers
- [ ] Users CRUD controller built
- [ ] Query parameters implemented
- [ ] Nested routes working
- [ ] Request/response customization done
- [ ] DTOs created

### Module 4: Providers
- [ ] UserService implemented
- [ ] Repository pattern applied
- [ ] Custom providers created
- [ ] Multi-layer architecture built

### Module 5: Modules
- [ ] Feature modules organized
- [ ] Shared module created
- [ ] Dynamic module implemented
- [ ] Monolith refactored

### Module 6: Core Fundamentals
- [ ] Exception filter created
- [ ] Unit tests written (good coverage)
- [ ] E2E tests implemented
- [ ] Configuration validated
- [ ] SOLID principles applied

### Module 7: Additional Fundamentals
- [ ] Middleware implemented
- [ ] Validation pipes working
- [ ] Auth guards protecting routes
- [ ] Interceptors transforming responses
- [ ] Custom decorators in use

### Module 8: Practical Application
- [ ] Task Management API complete
- [ ] Authentication working
- [ ] All CRUD operations tested
- [ ] Tests passing
- [ ] Documentation written

---

## 🏆 Completion Badges

Track your achievements:

- 🥉 **Bronze:** Completed Modules 1-3
- 🥈 **Silver:** Completed Modules 1-5
- 🥇 **Gold:** Completed Modules 1-7
- 💎 **Platinum:** Completed All Modules + Final Project

---

## 🆘 Getting Help

### When You're Stuck

1. **Read Error Messages Carefully**
   - They usually tell you what's wrong
   - Check line numbers in stack traces

2. **Review Module Content**
   - Go back to the theory
   - Check code examples

3. **Search Documentation**
   - [docs.nestjs.com](https://docs.nestjs.com)
   - Use search function

4. **Check Official Examples**
   - [NestJS GitHub Samples](https://github.com/nestjs/nest/tree/master/sample)

5. **Ask the Community**
   - Discord, Stack Overflow
   - Provide context and code

### Common Issues

**Port Already in Use**
```bash
# Find and kill process
lsof -i :3000
kill -9 <PID>
```

**Module Not Found**
```bash
# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

**TypeScript Errors**
```bash
# Clean build
npm run build
```

---

## 🚀 Next Steps After Completion

### Expand Your Knowledge
1. **Database Integration**
   - TypeORM or Prisma
   - PostgreSQL, MongoDB

2. **Authentication & Authorization**
   - OAuth2, SAML
   - Role-based access control

3. **Microservices**
   - Message queues (RabbitMQ, Kafka)
   - Service communication

4. **GraphQL**
   - @nestjs/graphql
   - Code-first vs Schema-first

5. **WebSockets**
   - @nestjs/websockets
   - Real-time applications

6. **Testing**
   - Advanced testing patterns
   - Mocking strategies

### Build Your Portfolio
- Share projects on GitHub
- Write blog posts about learnings
- Contribute to open source
- Build real-world applications

---

## 📝 Feedback

We'd love to hear about your experience:
- What exercises were most helpful?
- What could be improved?
- What additional topics would you like covered?

Create an issue or pull request to share feedback!

---

## 📜 License

These exercises are part of the NestJS Course and are intended for educational purposes.

---

**Happy Coding! 🎉**

[Back to Course Home](../README.md) | [View Course Outline](../course-outline.md)
