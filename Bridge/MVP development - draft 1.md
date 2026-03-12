```
bridgemvp/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── ...
├── src/
│   ├── components/
│   │   ├── Layout.jsx
│   │   └── DashboardCard.jsx
│   ├── pages/
│   │   ├── Dashboard.jsx
│   │   ├── InputForm.jsx
│   │   └── Settings.jsx
│   ├── App.js
│   ├── index.js
│   └── index.css
├── package.json
├── tailwind.config.js
└── postcss.config.js
```


# Prompt

I am building the frontend for a web-based platform that helps software teams and stakeholders visualize and communicate technical debt. The platform is in its MVP stage, with a focus on usability, simplicity, and scalability. Here’s the detailed information about the project and requirements: Project Context  
• Purpose: Bridge the communication gap between technical and non-technical stakeholders by providing a visual representation of technical debt and AI-generated plain-language summaries of complex technical issues.  
• Target Users: Software engineers, team leads, project managers, and executives who need to understand technical debt at a glance.  
• Primary Features:  
1. Data Input: Engineers input metrics (e.g., codebase size, test coverage, dependencies, etc.).  
2. Visualization Dashboard: Displays technical debt through charts and graphs (e.g., pie charts, line graphs, or bar graphs).  
3. Summarization: AI-generated text summaries explain the data in layman’s terms. Technical Requirements  
• Framework: Use React.js.  
• Styling: Use TailwindCSS for a modern and responsive design.  
• Charting Library: Use Chart.js or D3.js for creating interactive, visually appealing data visualizations.  
• State Management: Use React’s Context API or Redux to manage global states (e.g., user data, visualization options).  
• API Integration: Connect with a backend API that provides technical debt data and AI-generated summaries. Design and Functionality Goals  
1. Home Page / Dashboard:  
• A clean, minimalist interface that welcomes users.  
• An overview section summarizing the current state of technical debt (e.g., “X% of codebase flagged as high risk”).  
• Visual representations like:  
• A pie chart showing the proportion of different types of technical debt.  
• A bar graph tracking technical debt over time.  
• A text area or card showing an AI-generated summary.  
2. Input Form:  
• A simple, user-friendly form for engineers to input metrics like:  
• Codebase size (in lines of code).  
• Test coverage (percentage).  
• Number of unresolved issues or dependencies.  
• Include input validation (e.g., only numbers for metrics). 
- Software capitalization data - dev hours, projects, etc. 
3. Navigation:  
• A top navigation bar with links to:  
• Dashboard  
• Input Form  
• Settings  
• A responsive design that adapts to mobile and desktop users. We are focusing on desktop first. 
4. Responsiveness:  
• Ensure the platform is fully responsive and looks great on devices of all sizes (mobile, tablet, desktop).  
5. Accessibility:  
• Follow web accessibility standards (e.g., ARIA roles, color contrast) to ensure the platform is usable by all users. Specific Tasks for GPT  
1. Create React Components:  
• Develop reusable components like:  
• Navbar: A responsive top navigation bar.  
• DashboardCard: A card component to display summaries or key data points.  
• ChartContainer: A wrapper component for displaying charts.  
• InputForm: A form component with validation for user input.  
• Use functional components with hooks (e.g., useState, useEffect).  
2. Integrate Charting Library:  
• Set up a sample pie chart and line graph using Chart.js or D3.js.  
• Use mock data for now, but ensure the components can easily integrate with real data from an API.  
3. Styling with TailwindCSS:  
• Apply a modern, clean design to all components using TailwindCSS.  
• Use utility classes for spacing, colors, and typography.  
4. API Integration (Mock):  
• Simulate API calls using mock JSON data for:  
• Technical debt metrics.  
• AI-generated summaries.  
• Use fetch or axios to demonstrate how the frontend will communicate with the backend. Final Deliverable  
  
The output should include:  
1. A basic React application with:  
• A functional, responsive Dashboard page.  
• An Input Form page for data submission.  
• Reusable components styled with TailwindCSS.  
2. Interactive charts using mock data.  
3. A placeholder for API integration with mock API calls. Notes:  
• Ensure the code is modular and easy to maintain.  
• Provide comments explaining key sections of the code.  
• If necessary, suggest improvements or additional tools for better performance or scalability.