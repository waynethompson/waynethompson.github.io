---
layout: post
title:  "Understanding Linear Regression in C#: A Hands-On Implementation"
date:   2024-03-19 22:00:00 +1000
categories: [data, machine-learning]
tags: [data, machine-learning, csharp, statistics]
permalink: /blog/linear-regression-in-csharp/
---


Linear regression is one of the foundational algorithms in statistics and machine learning. It's used to discover relationships between variables and to make predictions. If you're exploring machine learning concepts or need a way to analyze trends in your C# applications, understanding linear regression is a great starting point.

**What is Linear Regression?**

In essence, linear regression attempts to draw the best-fitting straight line through a set of data points.  This line represents the underlying trend in the data. Imagine you have data about house prices based on their square footage – linear regression can help you model the relationship between these variables.

**Implementing Linear Regression in C#**

The basic implementation of an algorithm for linear regression in C# involves calculating the slope and y-intercept of the best-fit line. If you are interested in the math behind it read [Least squares regression](https://www.mathsisfun.com/data/least-squares-regression.html) on maths is fun.

Here's a simple example:

```csharp
using System;

namespace LinearRegression
{
    class LinearRegression
    {
        /// <summary>
        /// Calculates the slope of the best-fit line
        /// </summary>
        /// <param name="x">Array of x values</param>
        /// <param name="y">Array of y values</param>
        /// <returns>The slope of the regression line</returns>
        private double CalculateSlope(double[] x, double[] y)
        {
            double xMean = x.Average();
            double yMean = y.Average();

            double numerator = 0;
            double denominator = 0;

            for (int i = 0; i < x.Length; i++)
            {
                numerator += (x[i] - xMean) * (y[i] - yMean);
                denominator += (x[i] - xMean) * (x[i] - xMean);
            }

            return numerator / denominator;
        }

        /// <summary>
        /// Calculates the y-intercept of the best-fit line
        /// </summary>
        /// <param name="x">Array of x values</param>
        /// <param name="y">Array of y values</param>
        /// <param name="slope">The slope of the regression line</param>
        /// <returns>The y-intercept of the regression line</returns>
        private double CalculateIntercept(double[] x, double[] y, double slope)
        {
            double xMean = x.Average();
            double yMean = y.Average();

            return yMean - slope * xMean;
        }

        /// <summary>
        /// Predicts a y value based on a given x value
        /// </summary>
        /// <param name="x">The input x value</param>
        /// <param name="slope">The slope of the regression line</param>
        /// <param name="intercept">The y-intercept of the regression line</param>
        /// <returns>The predicted y value</returns>
        public double Predict(double x, double slope, double intercept)
        {
            return slope * x + intercept;
        }
    }
}
```

* **The `LinearRegression` Class:** This class encapsulates our algorithm:
* `CalculateSlope`: Finds the slope of our best-fit line.
* `CalculateIntercept`: Finds where the line crosses the y-axis.
* `Predict`: Uses the line's equation to predict y-values for new x-values.

**Using Our Algorithm**

```csharp
using System;
using LinearRegression; 

namespace Example
{
    class Program
    {
        static void Main(string[] args)
        {
            double[] x = { 1, 2, 3, 4, 5 };
            double[] y = { 2, 5, 7, 9, 12 };

            var regression = new LinearRegression();

            double slope = regression.CalculateSlope(x, y);
            double intercept = regression.CalculateIntercept(x, y, slope);

            Console.WriteLine("Equation: y = {0}x + {1}", slope.ToString("F2"), intercept.ToString("F2"));

            double prediction = regression.Predict(6, slope, intercept);
            Console.WriteLine("Prediction for x = 6: y = {0}", prediction.ToString("F2"));
        }
    }
}
```

In this example, we:

1. Define sample data (`x` and `y`).
2. Create a `LinearRegression` object.
3. Calculate the slope and intercept of the regression line.
4. Print the line's equation.
5. Make a prediction for a new input value.

**Beyond the Basics**

Keep in mind that this is a simplified implementation for educational purposes. Here are some things to consider for more robust work:

* **Data Validation:**  Ensure your data makes sense for linear regression (check for linear patterns).
* **Goodness of Fit:** Calculate metrics like R-squared to assess how well your line fits the data.
* **Advanced Libraries:** Explore libraries like ML.NET or Math.NET Numerics for more sophisticated statistical tools and machine learning models.

**Why Learn C# Linear Regression?**

* **Foundation for Machine Learning:** Linear regression is often a stepping stone into more complex machine learning algorithms.
* **Data Analysis in C# Applications:** If you work with data in C#, this allows you to analyze trends and make predictions right within your applications.

**Let's Get Practical**

Think about some datasets you might have access to.  Could you...

* Predict sales trends based on historical figures?
* Model the relationship between website traffic and advertising spend?
* Analyze sensor data for patterns?

Let me know if you'd like to explore specific use cases or dive deeper into improving the accuracy of your linear regression models! 


https://www.kaggle.com/datasets/himanshunakrani/student-study-hours?resource=download