---
title: CSS笔记
author: Zeeny
comments: false
date: 2024-03-02
updated: 2024-03-02
tags: [Html,CSS]
categories: [HTML,CSS]
summary: CSS笔记
---

# CSS固定表头不滚动

* css:
```
css:
position: sticky;
top: 0;
```

* html:
```
<style>
table {
  border: 1px solid;
  border-collapse: collapse;
  font-family: sans-serif;
  max-width: 100%;
  margin: 0 auto;
}
thead {
  background: black;
  color: white;
  position: sticky;
  text-align: left;
  top: 0;
}
tbody tr {
  border-bottom: 1px solid;
}
th,td {
  padding: 6px 12px;
}
td:nth-of-type(6),th:nth-of-type(6) {
  padding-left: 40px;
}
td:nth-of-type(4),th:nth-of-type(4),
td:nth-of-type(5),th:nth-of-type(5){
  text-align: right;
}
tr:nth-of-type(even) {
  background: #eee;
}
</style>

<table>
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Category</th>
      <th>Price</th>
      <th>Calories</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <!-- Row 1 -->
    <tr>
      <td>1</td>
      <td>Pizza</td>
      <td>Italian</td>
      <td>12.99</td>
      <td>800</td>
      <td>Classic Margherita</td>
    </tr>
    <!-- Row 2 -->
    <tr>
      <td>2</td>
      <td>Burger</td>
      <td>American</td>
      <td>8.99</td>
      <td>600</td>
      <td>Cheeseburger with fries</td>
    </tr>
    <!-- Row 3 -->
    <tr>
      <td>3</td>
      <td>Sushi</td>
      <td>Japanese</td>
      <td>22.99</td>
      <td>500</td>
      <td>Assorted sushi rolls</td>
    </tr>
    <!-- Row 4 -->
    <tr>
      <td>4</td>
      <td>Tacos</td>
      <td>Mexican</td>
      <td>10.50</td>
      <td>450</td>
      <td>Beef and chicken tacos</td>
    </tr>
    <!-- Row 5 -->
    <tr>
      <td>5</td>
      <td>Salad</td>
      <td>Healthy</td>
      <td>7.99</td>
      <td>300</td>
      <td>Fresh garden salad</td>
    </tr>   
    <!-- Row 6 -->
    <tr>
      <td>6</td>
      <td>Pasta</td>
      <td>Italian</td>
      <td>14.50</td>
      <td>700</td>
      <td>Spaghetti Bolognese</td>
    </tr>
    <!-- Row 7 -->
    <tr>
      <td>7</td>
      <td>Chicken Curry</td>
      <td>Indian</td>
      <td>12.99</td>
      <td>550</td>
      <td>Authentic Indian curry</td>
    </tr>
    <!-- Row 8 -->
    <tr>
      <td>8</td>
      <td>Sandwich</td>
      <td>American</td>
      <td>6.99</td>
      <td>400</td>
      <td>Club sandwich</td>
    </tr>
    <!-- Row 9 -->
    <tr>
      <td>9</td>
      <td>Sushi Bowl</td>
      <td>Japanese</td>
      <td>18.99</td>
      <td>600</td>
      <td>Chirashi bowl</td>
    </tr>
    <!-- Row 10 -->
    <tr>
        <td>10</td>
        <td>Shrimp Scampi</td>
        <td>Italian</td>
        <td>16.99</td>
        <td>550</td>
        <td>Garlic butter shrimp pasta</td>
    </tr>
    <!-- Row 11 -->
    <tr>
        <td>11</td>
        <td>Fish and Chips</td>
        <td>British</td>
        <td>13.50</td>
        <td>700</td>
        <td>Classic British dish</td>
    </tr>
    <!-- Row 12 -->
    <tr>
        <td>12</td>
        <td>Ramen</td>
        <td>Japanese</td>
        <td>11.99</td>
        <td>500</td>
        <td>Tonkotsu ramen</td>
    </tr>
    <!-- Row 13 -->
    <tr>
        <td>13</td>
        <td>Caesar Salad</td>
        <td>Healthy</td>
        <td>8.50</td>
        <td>350</td>
        <td>Classic Caesar salad</td>
    </tr>
    <!-- Row 14 -->
    <tr>
        <td>14</td>
        <td>Steak</td>
        <td>American</td>
        <td>25.99</td>
        <td>800</td>
        <td>Grilled sirloin steak</td>
    </tr>
    <!-- Row 15 -->
    <tr>
        <td>15</td>
        <td>Pho</td>
        <td>Vietnamese</td>
        <td>10.99</td>
        <td>450</td>
        <td>Beef pho</td>
    </tr>
    <!-- Row 16 -->
    <tr>
        <td>16</td>
        <td>Lasagna</td>
        <td>Italian</td>
        <td>16.50</td>
        <td>700</td>
        <td>Homemade meat lasagna</td>
    </tr>
    <!-- Row 17 -->
    <tr>
        <td>17</td>
        <td>Fajitas</td>
        <td>Mexican</td>
        <td>14.99</td>
        <td>600</td>
        <td>Chicken and beef fajitas</td>
    </tr>
    <!-- Row 18 -->
    <tr>
        <td>18</td>
        <td>Caprese Salad</td>
        <td>Italian</td>
        <td>9.99</td>
        <td>350</td>
        <td>Tomato, mozzarella, and basil</td>
    </tr>
    <!-- Row 19 -->
    <tr>
      <td>19</td>
      <td>Pad Thai</td>
      <td>Thai</td>
      <td>12.50</td>
      <td>500</td>
      <td>Traditional Pad Thai</td>
    </tr>
    <!-- Row 20 -->
    <tr>
      <td>20</td>
      <td>Chicken Alfredo</td>
      <td>Italian</td>
      <td>15.99</td>
      <td>600</td>
      <td>Creamy chicken Alfredo pasta</td>
    </tr>       
    <!-- Row 21 -->
    <tr>
      <td>21</td>
      <td>Vegetable Stir Fry</td>
      <td>Chinese</td>
      <td>11.50</td>
      <td>400</td>
      <td>Assorted vegetables stir-fried</td>
    </tr>
    <!-- Row 22 -->
    <tr>
      <td>22</td>
      <td>Quinoa Bowl</td>
      <td>Healthy</td>
      <td>9.99</td>
      <td>350</td>
      <td>Quinoa with mixed vegetables</td>
    </tr>
    <!-- Row 23 -->
    <tr>
      <td>23</td>
      <td>Tom Yum Soup</td>
      <td>Thai</td>
      <td>8.50</td>
      <td>250</td>
      <td>Spicy and sour soup with shrimp</td>
    </tr>
    <!-- Row 24 -->
    <tr>
      <td>24</td>
      <td>Bruschetta</td>
      <td>Italian</td>
      <td>7.99</td>
      <td>300</td>
      <td>Tomato and basil bruschetta</td>
    </tr>
    <!-- Row 25 -->
    <tr>
      <td>25</td>
      <td>Teriyaki Salmon</td>
      <td>Japanese</td>
      <td>18.50</td>
      <td>600</td>
      <td>Grilled teriyaki-glazed salmon</td>
    </tr>
    <!-- Row 26 -->
    <tr>
      <td>26</td>
      <td>Chicken Shawarma</td>
      <td>Middle Eastern</td>
      <td>10.99</td>
      <td>500</td>
      <td>Marinated and grilled chicken</td>
    </tr>
    <!-- Row 27 -->
    <tr>
      <td>27</td>
      <td>Cheese Platter</td>
      <td>Appetizer</td>
      <td>14.99</td>
      <td>400</td>
      <td>Assorted cheeses with crackers</td>
    </tr>
    <!-- Row 28 -->
    <tr>
      <td>28</td>
      <td>Mango Sticky Rice</td>
      <td>Thai</td>
      <td>6.50</td>
      <td>300</td>
      <td>Sweet mango with sticky rice</td>
    </tr>
    <!-- Row 29 -->
    <tr>
      <td>29</td>
      <td>Spinach Artichoke Dip</td>
      <td>Appetizer</td>
      <td>8.99</td>
      <td>350</td>
      <td>Creamy spinach and artichoke dip</td>
    </tr>
    <!-- Row 30 -->
    <tr>
      <td>30</td>
      <td>Cherry Cheesecake</td>
      <td>Dessert</td>
      <td>7.50</td>
      <td>450</td>
      <td>Classic cherry-topped cheesecake</td>
    </tr>
    <!-- Row 31 -->
    <tr>
      <td>31</td>
      <td>Chicken Enchiladas</td>
      <td>Mexican</td>
      <td>12.99</td>
      <td>550</td>
      <td>Shredded chicken enchiladas</td>
    </tr>
    <!-- Row 32 -->
    <tr>
      <td>32</td>
      <td>Beef Stroganoff</td>
      <td>Russian</td>
      <td>15.50</td>
      <td>650</td>
      <td>Creamy beef stroganoff with mushrooms</td>
    </tr>
    <!-- Row 33 -->
    <tr>
      <td>33</td>
      <td>Chicken Caesar Wrap</td>
      <td>American</td>
      <td>9.50</td>
      <td>400</td>
      <td>Grilled chicken in a Caesar wrap</td>
    </tr>
    <!-- Row 34 -->
    <tr>
      <td>34</td>
      <td>Avocado Toast</td>
      <td>Healthy</td>
      <td>6.99</td>
      <td>350</td>
      <td>Sliced avocado on toasted bread</td>
    </tr>
    <!-- Row 35 -->
    <tr>
      <td>35</td>
      <td>Lemon Garlic Shrimp</td>
      <td>Seafood</td>
      <td>16.99</td>
      <td>500</td>
      <td>Grilled shrimp with lemon and garlic</td>
    </tr>
    <!-- Row 36 -->
    <tr>
      <td>36</td>
      <td>Tomato Basil Soup</td>
      <td>Soup</td>
      <td>7.50</td>
      <td>300</td>
      <td>Homemade tomato basil soup</td>
    </tr>
    <!-- Row 37 -->
    <tr>
      <td>37</td>
      <td>Cajun Jambalaya</td>
      <td>Cajun</td>
      <td>14.50</td>
      <td>600</td>
      <td>Spicy Cajun rice dish with meat and vegetables</td>
    </tr>
    <!-- Row 38 -->
    <tr>
      <td>38</td>
      <td>Chocolate Fondue</td>
      <td>Dessert</td>
      <td>11.99</td>
      <td>450</td>
      <td>Assorted fruits dipped in chocolate</td>
    </tr>
    <!-- Row 39 -->
    <tr>
      <td>39</td>
      <td>Eggplant Parmesan</td>
      <td>Italian</td>
      <td>13.99</td>
      <td>550</td>
      <td>Baked eggplant with marinara and cheese</td>
    </tr>
    <!-- Row 40 -->
    <tr>
      <td>40</td>
      <td>Chicken Noodle Soup</td>
      <td>Soup</td>
      <td>8.50</td>
      <td>350</td>
      <td>Classic chicken noodle soup</td>
    </tr>
  </tbody>
</table>
```
### Please Make Your Table Headings Sticky
[Please Make Your Table Headings Sticky](https://btxx.org/posts/Please_Make_Your_Table_Headings_Sticky/)
[Tables with Scrollable Headers](https://codepen.io/bradleytaunt/pen/bGZyJBj)