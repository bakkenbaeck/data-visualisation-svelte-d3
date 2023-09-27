# Interactive Data Visualization with Svelte and D3

![animated-bar-chart](/src/assets/animated-bar-chart.gif)

## Table of Contents

- [Installation](#installation)
- [Running the Project](#running-the-project)
- [Dependencies](#dependencies)
- [Charts](#available-charts)

## Installation

To run this project locally, follow these steps:

1. Clone the repository to your local machine:
    ```bash
       git clone https://github.com/your-username/data-visualisation-svelte-d3.git
    ```

2. Change to the project directory:
    ```bash
       cd data-visualisation-svelte-d3
    ```

3. Install the project dependencies:
    ```bash
       npm install
    ```

## Running the Project
After successfully installing the dependencies, you can run the project with the following commands:

- Start the development server:
    ```bash
       npm run dev
    ```
    This will launch the project in development mode, and you can access it in your browser at http://localhost:3000.


- Build the project for production:
    ```bash
       npm run build
    ```
    This command will generate optimized production-ready files in the dist folder.


- Preview the production build:
    ```bash
       npm run preview
    ```
  Use this command to preview the production build locally before deploying it.

## Dependencies
This project relies on several dependencies to create interactive data visualizations:

- __Svelte__: A JavaScript framework for building user interfaces.
- __Vite__: A fast development build tool.
- __D3.js__: A powerful library for creating data-driven visualizations.
- __Lodash__: A utility library for handling data transformations.
- __TopoJSON__: A format for encoding geographic data.
- __Versor__: A library for 3D vector math.


## Available Charts

The project includes various interactive charts:

1. __Bar Chart__: Displays a simple bar chart without interactivity or animation.
2. __Interactive Bar Chart__: On bars hover interactively shows value and changing color of the bar.
3. __Animated Bar Chart__: Features animated transitions in the bar chart.
4. __Bubble Chart__: Creates an interactive data visualization that displays circles on a chart based on income, life expectancy, and population data for different countries over time.
5. __US Map__: Shows a map of the United States with data visualization.
6. __World Globe__: Presents an interactive 3D globe for global data exploration.

You can switch between these charts using the navigation menu in the application. Each chart provides a unique data visualization experience.
