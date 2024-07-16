---
layout: post
title:  "A React Frontend Code Review Checklist"
date:   2024-07-13 12:00:00 +1000
categories: [engineering, react]
tags: [react, code review, technical debt]
permalink: /blog/React-Code-Review-Checklist/
---

For my own uses I thought I'd compile a checklist for code reviews for react frontends. So in this post, we'll go through a comprehensive checklist to ensure your React frontend code is up to par.

1. Component Structure
   - Are components properly divided and modular?
   - Is there a clear separation of concerns?
   - Are components kept reasonably small and focused?
   - Components over 200 lines should be broken down into smaller components.
   - JSX markup shouold be no more than 50 lines.

2. Props and State Management
   - Are props being used correctly and consistently?
   - No unused props are being passed.
   - Is state managed efficiently? (Consider using hooks like useState or useReducer)
   - Are there any unnecessary re-renders due to improper state management?

3. Performance Optimization
   - Are heavy computations memoized using useMemo or useCallback?
   - Is React.memo used appropriately for component memoization?
   - Are large lists rendered efficiently (e.g., using virtualization)?

4. Code Style and Formatting
   - Does the code follow the project's style guide?
   - Is the code consistently formatted?
   - Are naming conventions clear and consistent?
   - Every function should have comments explaining its purpose and inputs/outputs.
   - Code should have no linter errors.
   - Naming conventions followed for variables, file names, translations.
   - No harcoded values or magic numbers/strings. Use constant values.
   - No console.log or debugger statements left in the code.
   - Group similar values under an enumeration (enum) or object.

5. Error Handling
   - Are errors properly caught and handled?
   - Is there appropriate error boundary implementation?
   - Are user-facing error messages clear and helpful?

6. Testing
   - Are there sufficient unit tests for components and functions?
   - Are critical user flows covered by integration tests?
   - Is there proper mocking of external dependencies in tests?

7. Accessibility (a11y)
   - Are semantic HTML elements used appropriately?
   - Do all interactive elements have proper ARIA attributes?
   - Is the tab order logical and does keyboard navigation work as expected?

8. Security
   - Is user input properly sanitized?
   - Are there any potential XSS vulnerabilities?
   - Is sensitive data handled securely?

9. Dependencies
   - Are all dependencies necessary and up-to-date?
   - Are there any potential conflicts or security issues with dependencies?

10. Documentation
    - Is the code self-documenting where possible?
    - Are complex logic or algorithms properly commented?
    - Is there sufficient documentation for the component API?

11. React Best Practices
    - Are hooks rules followed (only called at the top level of functional components)?
    - Is the key prop used correctly in lists?
    - Are side effects properly managed with useEffect?

12. Code Duplication
    - Is there any unnecessary code duplication?
    - Can any repeated logic be abstracted into custom hooks or utility functions?

13. Responsiveness
    - Does the UI adapt well to different screen sizes?
    - Are media queries or responsive design techniques used effectively?

14. Internationalization (i18n)
    - Are all user-facing strings properly internationalized?

15. Performance Profiling
    - Have you run the React DevTools profiler to check for performance issues?
    - Are there any unnecessary rerenders or slow components?

This checklist is by no means exhaustive, but it covers many essential aspects of React frontend development. By following this guide during code reviews, you can ensure that your React applications are maintainable, performant, and user-friendly.