
# 💡 Data Science TIL

> **Today I Learned**

<br>

---

## 🛠 Tech Stack

<div align="center">

### Language

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)

### Data Analysis

![NumPy](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557C?style=flat-square&logo=python&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0?style=flat-square&logo=python&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=flat-square&logo=scipy&logoColor=white)


### Statistics

![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=flat-square&logo=scipy&logoColor=white)
![statsmodels](https://img.shields.io/badge/statsmodels-4051B5?style=flat-square&logo=python&logoColor=white)


### Machine Learning

![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=white)

### Deep Learning

![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white)

### Database

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![DBeaver](https://img.shields.io/badge/DBeaver-382923?style=flat-square&logo=dbeaver&logoColor=white)

### Generative AI

![Claude](https://img.shields.io/badge/Claude-D97757?style=flat-square&logo=claude&logoColor=white)
![ChatGPT](https://img.shields.io/badge/ChatGPT-000000?style=flat-square&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNTYgMjYwIj48cGF0aCBmaWxsPSIjZmZmIiBkPSJNMjM5LjE4NCAxMDYuMjAzYTY0LjcxNiA2NC43MTYgMCAwIDAtNS41NzYtNTMuMTAzQzIxOS40NTIgMjguNDU5IDE5MSAxNS43ODQgMTYzLjIxMyAyMS43NEE2NS41ODYgNjUuNTg2IDAgMCAwIDUyLjA5NiA0NS4yMmE2NC43MTYgNjQuNzE2IDAgMCAwLTQzLjIzIDMxLjM2Yy0xNC4zMSAyNC42MDItMTEuMDYxIDU1LjYzNCA4LjAzMyA3Ni43NGE2NC42NjUgNjQuNjY1IDAgMCAwIDUuNTI1IDUzLjEwMmMxNC4xNzQgMjQuNjUgNDIuNjQ0IDM3LjMyNCA3MC40NDYgMzEuMzZhNjQuNzIgNjQuNzIgMCAwIDAgNDguNzU0IDIxLjc0NGMyOC40ODEuMDI1IDUzLjcxNC0xOC4zNjEgNjIuNDE0LTQ1LjQ4MWE2NC43NjcgNjQuNzY3IDAgMCAwIDQzLjIyOS0zMS4zNmMxNC4xMzctMjQuNTU4IDEwLjg3NS01NS40MjMtOC4wODMtNzYuNDgzWm0tOTcuNTYgMTM2LjMzOGE0OC4zOTcgNDguMzk3IDAgMCAxLTMxLjEwNS0xMS4yNTVsMS41MzUtLjg3IDUxLjY3LTI5LjgyNWE4LjU5NSA4LjU5NSAwIDAgMCA0LjI0Ny03LjM2N3YtNzIuODVsMjEuODQ1IDEyLjYzNmMuMjE4LjExMS4zNy4zMi40MDkuNTYzdjYwLjM2N2MtLjA1NiAyNi44MTgtMjEuNzgzIDQ4LjU0NS00OC42MDEgNDguNjAxWm0tMTA0LjQ2Ni00NC42MWE0OC4zNDUgNDguMzQ1IDAgMCAxLTUuNzgxLTMyLjU4OWwxLjUzNC45MjEgNTEuNzIyIDI5LjgyNmE4LjMzOSA4LjMzOSAwIDAgMCA4LjQ0MSAwbDYzLjE4MS0zNi40MjV2MjUuMjIxYS44Ny44NyAwIDAgMS0uMzU4LjY2NWwtNTIuMzM1IDMwLjE4NGMtMjMuMjU3IDEzLjM5OC01Mi45NyA1LjQzMS02Ni40MDQtMTcuODAzWk0yMy41NDkgODUuMzhhNDguNDk5IDQ4LjQ5OSAwIDAgMSAyNS41OC0yMS4zMzN2NjEuMzlhOC4yODggOC4yODggMCAwIDAgNC4xOTUgNy4zMTZsNjIuODc0IDM2LjI3Mi0yMS44NDUgMTIuNjM2YS44MTkuODE5IDAgMCAxLS43NjcgMEw0MS4zNTMgMTUxLjUzYy0yMy4yMTEtMTMuNDU0LTMxLjE3MS00My4xNDQtMTcuODA0LTY2LjQwNXYuMjU2Wm0xNzkuNDY2IDQxLjY5NS02My4wOC0zNi42M0wxNjEuNzMgNzcuODZhLjgxOS44MTkgMCAwIDEgLjc2OCAwbDUyLjIzMyAzMC4xODRhNDguNiA0OC42IDAgMCAxLTcuMzE2IDg3LjYzNXYtNjEuMzkxYTguNTQ0IDguNTQ0IDAgMCAwLTQuNC03LjIxM1ptMjEuNzQyLTMyLjY5LTEuNTM1LS45MjItNTEuNjE5LTMwLjA4MWE4LjM5IDguMzkgMCAwIDAtOC40OTIgMEw5OS45OCA5OS44MDhWNzQuNTg3YS43MTYuNzE2IDAgMCAxIC4zMDctLjY2NWw1Mi4yMzMtMzAuMTMzYTQ4LjY1MiA0OC42NTIgMCAwIDEgNzIuMjM2IDUwLjM5MXYuMjA1Wk04OC4wNjEgMTM5LjA5N2wtMjEuODQ1LTEyLjU4NWEuODcuODcgMCAwIDEtLjQxLS42MTRWNjUuNjg1YTQ4LjY1MiA0OC42NTIgMCAwIDEgNzkuNzU3LTM3LjM0NmwtMS41MzUuODctNTEuNjcgMjkuODI1YTguNTk1IDguNTk1IDAgMCAwLTQuMjQ2IDcuMzY3bC0uMDUxIDcyLjY5N1ptMTEuODY4LTI1LjU4IDI4LjEzOC0xNi4yMTcgMjguMTg4IDE2LjIxOHYzMi40MzRsLTI4LjA4NiAxNi4yMTgtMjguMTg4LTE2LjIxOC0uMDUyLTMyLjQzNFoiLz48L3N2Zz4%3D)


### Tools

![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white)
[![VS Code](https://img.shields.io/badge/VS_Code-007ACC?style=flat&logo=data:image/svg%2Bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMjggMTI4Ij48cGF0aCBmaWxsPSIjZmZmZmZmIiBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTkwLjc2NyAxMjcuMTI2YTcuOTY4IDcuOTY4IDAgMCAwIDYuMzUtLjI0NGwyNi4zNTMtMTIuNjgxYTggOCAwIDAgMCA0LjUzLTcuMjA5VjIxLjAwOWE4IDggMCAwIDAtNC41My03LjIxTDk3LjExNyAxLjEyYTcuOTcgNy45NyAwIDAgMC05LjA5MyAxLjU0OGwtNTAuNDUgNDYuMDI2TDE1LjYgMzIuMDEzYTUuMzI4IDUuMzI4IDAgMCAwLTYuODA3LjMwMmwtNy4wNDggNi40MTFhNS4zMzUgNS4zMzUgMCAwIDAtLjAwNiA3Ljg4OEwyMC43OTYgNjQgMS43NCA4MS4zODdhNS4zMzYgNS4zMzYgMCAwIDAgLjAwNiA3Ljg4N2w3LjA0OCA2LjQxMWE1LjMyNyA1LjMyNyAwIDAgMCA2LjgwNy4zMDNsMjEuOTc0LTE2LjY4IDUwLjQ1IDQ2LjAyNWE3Ljk2IDcuOTYgMCAwIDAgMi43NDMgMS43OTNabTUuMjUyLTkyLjE4M0w1Ny43NCA2NGwzOC4yOCAyOS4wNThWMzQuOTQzWiIvPjwvc3ZnPgo=&logoColor=white)](https://code.visualstudio.com/)
</div>

<br>

---

# 🖥️ TIL

