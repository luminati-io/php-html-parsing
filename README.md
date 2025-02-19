# Parsing HTML with PHP

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide examines three PHP HTML parsing techniques and compares their strengths and differences:

- [Parsing HTML with PHP](#parsing-html-with-php)
- [Why Parse HTML in PHP?](#why-parse-html-in-php)
- [Prerequisites](#prerequisites)
- [HTML Retrieval in PHP](#html-retrieval-in-php)
- [HTML Parsing in PHP: 3 Approaches](#html-parsing-in-php-3-approaches)
  - [Approach #1: With `Dom\HTMLDocument`](#approach-1-with-domhtmldocument)
  - [Approach #2: Using Simple HTML DOM Parser](#approach-2-using-simple-html-dom-parser)
  - [Approach #3: Using Symfony’s DomCrawler Component](#approach-3-using-symfonys-domcrawler-component)
- [Parsing HTML in PHP: Comparison Table](#parsing-html-in-php-comparison-table)
- [Conclusion](#conclusion)

## Why Parse HTML in PHP?

HTML Parsing in PHP involves converting HTML content into its DOM ([Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)) structure. Once in the DOM format, you can easily navigate and manipulate the HTML content.

In particular, the top reasons to parse HTML in PHP are:

- **Data extraction**: Retrieve specific content from web pages, including text or attributes from HTML elements.  
- **Automation**: Streamline tasks such as content scraping, reporting, and data aggregation from HTML.  
- **Server-side HTML handling**: Parse and manipulate HTML to clean, format, or modify web content before rendering it in your application.  


## Prerequisites

Before you start coding, make sure you have [PHP 8.4+](https://www.php.net/releases/8.4/en.php) installed on your machine. You can verify this by running the following command:

```bash
php -v
```

The output should look something like this:

```
PHP 8.4.3 (cli) (built: Jan 19 2025 14:20:58) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.4.3, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.3, Copyright (c), by Zend Technologies
```

Next, initialize a Composer project to make dependency management easier. If Composer is not installed on your system, [download it](https://getcomposer.org/download/) and follow the installation instructions.

First, create a new folder for your PHP HTML project:

```bash
mkdir php-html-parser
```

Navigate to the folder in your terminal and initialize a Composer project inside it using the `composer init` command:

```bash
composer init
```

During this process, you'll be asked a few questions. The default answers are sufficient, but you can provide more specific details to tailor the setup for your PHP HTML parsing project if needed.

Next, open the project folder in your favorite IDE. [Visual Studio Code with the PHP extension](https://code.visualstudio.com/docs/languages/php) or [IntelliJ WebStorm](https://www.jetbrains.com/webstorm/) are good choices for PHP development.

Now, add an empty `index.php` file to the project folder. Your project structure should now look like this:

```
php-html-parser/
  ├── vendor/
  ├── composer.json
  └── index.php
```

Open `index.php` and add the following code to initialize your project:

```php
<?php

require_once __DIR__ . "/vendor/autoload.php";

// scraping logic...
```

Run your script with this command:

```bash
php index.php
```

## HTML Retrieval in PHP

Before parsing HTML in PHP, you need some HTML to parse. In this section, we will see two different approaches to accessing HTML content in PHP. We suggest to read our [guide on web scraping with PHP](https://brightdata.com/blog/how-tos/web-scraping-php) too.

### With CURL

PHP natively supports cURL, a popular HTTP client used to perform HTTP requests. [Enable the cURL extension](https://www.php.net/manual/en/book.curl.php) or install it on Ubuntu Linux with:

```bash
sudo apt-get install php8.4-curl
```

You can use cURL to send an HTTP GET request to an online server and retrieve the HTML document returned by the server. This example script makes a simple GET request and retrieves HTML content:

```bash
// initialize cURL session
$ch = curl_init();

// set the URL you want to make a GET request to
curl_setopt($ch, CURLOPT_URL, "https://www.scrapethissite.com/pages/forms/?per_page=100");

// return the response instead of outputting it
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

// execute the cURL request and store the result in $response
$html = curl_exec($ch);

// close the cURL session
curl_close($ch);

// output the HTML response
echo $html;
```

Add the above code snippet to `index.php` and launch it. It will produce the following HTML code:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hockey Teams: Forms, Searching and Pagination | Scrape This Site | A public sandbox for learning web scraping</title>
    <link rel="icon" type="image/png" href="/static/images/scraper-icon.png" />
    <!-- Omitted for brevity... -->
</html>
```

### From a File

Let's assume you have a file named `index.html` that contains the HTML of the “Hockey Teams” page from [Scrape This Site](https://www.scrapethissite.com/pages/forms/?per_page=100), which was previously retrieved using cURL:

![The index.html file in the project folder](https://github.com/luminati-io/php-html-parsing/blob/main/Images/The-index.html-file-in-the-project-folder-2048x1216.png)

## HTML Parsing in PHP: 3 Approaches

This section explains using three different libraries to parse HTML in PHP:

1. Using `Dom\HTMLDocument` for vanilla PHP
2. Using the Simple HTML DOM Parser library
3. Using Symfony’s `DomCrawler` component

In all three cases, you parse the HTML from the local `index.html` file to select all hockey team entries on the page and extract data from them:

![The table on the target page](https://github.com/luminati-io/php-html-parsing/blob/main/Images/The-table-on-the-target-page-2048x1107.png)

The final result will be a list of scraped hockey team entries containing the following details:

- Team Name
- Year
- Wins
- Losses
- Win %
- Goals For (GF)
- Goals Against (GA)
- Goal Difference

You can extract them from the HTML table with this structure:

![The HTML DOM structure of the table's rows](https://github.com/luminati-io/php-html-parsing/blob/main/Images/The-HTML-DOM-structure-of-the-tables-rows-2048x1134.png)

Each column in a table row has a specific class, allowing you to extract data by selecting elements with their class as a CSS selector and retrieving their content through their text.

## Approach #1: With Dom\\HTMLDocument

PHP 8.4+ comes with a built-in [`Dom\HTMLDocument`](https://www.php.net/manual/en/class.dom-htmldocument.php) class. This represents an HTML document and allows you to parse HTML content and navigate the DOM tree.

### Step #1: Installation and Set Up

`Dom\HTMLDocument` is part of the [Standard PHP Library](https://www.php.net/manual/en/book.spl.php). Still, you need to enable the [DOM extension](https://www.php.net/manual/en/intro.dom.php) or install it with this Linux command to use it:

```bash
sudo apt-get install php-dom
```

### Step #2: HTML Parsing

You can parse the HTML string as below:

```php
$dom = \DOM\HTMLDocument::createFromString($html);
```

You can parse the `index.html` file with:

```php
$dom = \DOM\HTMLDocument::createFromFile("./index.html");
```

`$dom` is a [`Dom\HTMLDocument`](https://www.php.net/manual/en/class.dom-htmldocument.php) object that exposes the methods you need for data parsing.

### Step #3: Data Parsing

You can select all hockey team entries using `\DOM\HTMLDocument` with the following approach:

```php
// select each row on the page
$table = $dom->getElementsByTagName("table")->item(0);
$rows = $table->getElementsByTagName("tr");

// iterate through each row and extract data
foreach ($rows as $row) {
  $cells = $row->getElementsByTagName("td");

  // extracting the data from each column
  $team = trim($cells->item(0)->textContent);
  $year = trim($cells->item(1)->textContent);
  $wins = trim($cells->item(2)->textContent);
  $losses = trim($cells->item(3)->textContent);
  $win_pct = trim($cells->item(5)->textContent);
  $goals_for = trim($cells->item(6)->textContent);
  $goals_against = trim($cells->item(7)->textContent);
  $goal_diff = trim($cells->item(8)->textContent);

  // create an array for the scraped team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print ("\n");
}
```

`\DOM\HTMLDocument` does not offer advanced query methods. So you have to rely on methods like `getElementsByTagName()` and manual iteration.

Here is a breakdown of the methods used:

- `getElementsByTagName()`: Retrieve all elements of a given tag (like `<table>`, `<tr>`, or `<td>`) within the document.
- `item()`: Return an individual element from a list of elements returned by `getElementsByTagName()`.
- `textContent`: This property gives the raw text content of an element, allowing you to extract the visible data (like the team name, year, etc.).

We also used [`trim()`](https://www.php.net/manual/en/function.trim.php) to remove extra whitespace before and after the text content for cleaner data.

When added to `index.php`, the above snippet will produce this result:

```php
Array
(
    [team] => Boston Bruins
    [year] => 1990
    [wins] => 44
    [losses] => 24
    [win_pct] => 0.55
    [goals_for] => 299
    [goals_against] => 264
    [goal_diff] => 35
)

// omitted for brevity...

Array
(
    [team] => Detroit Red Wings
    [year] => 1994
    [wins] => 33
    [losses] => 11
    [win_pct] => 0.688
    [goals_for] => 180
    [goals_against] => 117
    [goal_diff] => 63
) 
```

## Approach #2: Using Simple HTML DOM Parser

[Simple HTML DOM Parser](https://github.com/voku/simple_html_dom) is a lightweight PHP library that makes it easy to parse and manipulate HTML content.

### Step #1: Installation and Set Up

You can install Simple HTML Dom Parser via Composer with this command:

```php
composer require voku/simple_html_dom
```

Alternatively, you can manually download and include the `simple_html_dom.php` file in your project.

Then, import it in `index.php` with this line of code:

```php
use voku\helper\HtmlDomParser;
```

### Step #2: HTML Parsing

To parse an HTML string, use the `file_get_html()` method:

```php
$dom = HtmlDomParser::str_get_html($html);
```

For parsing `index.html`, write `file_get_html()` instead:

```php
$dom = HtmlDomParser::file_get_html($str);
```

This will load the HTML content into a `$dom` object, which allows you to navigate the DOM easily.

### Step #3: Data Parsing

Extract the hockey team data from the HTML using Simple HTML DOM Parser:

```php
// find all rows in the table
$rows = $dom->findMulti("table tr.team");

// loop through each row to extract the data
foreach ($rows as $row) {
  // extract data using CSS selectors
  $team_element = $row->findOne(".name");
  $team = trim($team_element->plaintext);

  $year_element = $row->findOne(".year");
  $year = trim($year_element->plaintext);

  $wins_element = $row->findOne(".wins");
  $wins = trim($wins_element->plaintext);

  $losses_element = $row->findOne(".losses");
  $losses = trim($losses_element->plaintext);

  $win_pct_element = $row->findOne(".pct");
  $win_pct = trim($win_pct_element->plaintext);

  $goals_for_element = $row->findOne(".gf");
  $goals_for = trim($goals_for_element->plaintext);

  $goals_against_element = $row->findOne(".ga");
  $goals_against = trim(string: $goals_against_element->plaintext);

  $goal_diff_element = $row->findOne(".diff");
  $goal_diff = trim(string: $goal_diff_element->plaintext);

  // create an array with the extracted team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print("\n");
}
```

The Simple HTML DOM Parser features used above are:

- `findMulti()`: Select all elements identified by the given CSS selector.
- `findOne()`: Locate the first element matching the given CSS selector.
- `plaintext`: An attribute to get the raw text content inside an HTML element.

This time, we applied CSS selectors with a more comprehensive and robust logic. However, the result remains the same as in the initial PHP HTML parsing approach.

## Approach #3: Using Symfony’s DomCrawler Component

[Symfony’s `DomCrawler` component](https://symfony.com/doc/current/components/dom_crawler.html) provides an easy way to parse HTML documents and extract data from them.

> **Note**:
> The component is part of the [Symfony framework](https://symfony.com/) but can also be used standalone, as we will do in this section.

### Step #1: Installation and Set Up

Install Symfony’s `DomCrawler` component with this Composer command:

```bash
composer require symfony/dom-crawler
```

Then, import it in the `index.php` file:

```php
use Symfony\Component\DomCrawler\Crawler;
```

### Step #2: HTML Parsing

To parse an HTML string, create a [`Crawler`](https://github.com/symfony/symfony/blob/7.2/src/Symfony/Component/DomCrawler/Crawler.php) instance with the `html()` method:

```php
$crawler = new Crawler($html);
```

For parsing a file, use `file_get_contents()` and create the `Crawler` instance:

```php
$crawler = new Crawler(file_get_contents("./index.html"));
```

The above lines will load the HTML content into the `$crawler` object, which provides easy methods to traverse and extract data.

### Step #3: Data Parsing

Extract the hockey team data using the `DomCrawler` component:

```php
// select all rows within the table
$rows = $crawler->filter("table tr.team");

// loop through each row to extract the data
$rows->each(function ($row, $i) {
  // extract data using CSS selectors
  $team_element = $row->filter(".name");
  $team = trim($team_element->text());

  $year_element = $row->filter(".year");
  $year = trim($year_element->text());

  $wins_element = $row->filter(".wins");
  $wins = trim($wins_element->text());

  $losses_element = $row->filter(".losses");
  $losses = trim($losses_element->text());

  $win_pct_element = $row->filter(".pct");
  $win_pct = trim($win_pct_element->text());

  $goals_for_element = $row->filter(".gf");
  $goals_for = trim($goals_for_element->text());

  $goals_against_element = $row->filter(".ga");
  $goals_against = trim($goals_against_element->text());

  $goal_diff_element = $row->filter(".diff");
  $goal_diff = trim($goal_diff_element->text());

  // create an array with the extracted team data
  $team_data = [
    "team" => $team,
    "year" => $year,
    "wins" => $wins,
    "losses" => $losses,
    "win_pct" => $win_pct,
    "goals_for" => $goals_for,
    "goals_against" => $goals_against,
    "goal_diff" => $goal_diff
  ];

  // print the scraped team data
  print_r($team_data);
  print ("\n");
});
```

The `DomCrawler` methods used are:

- `each()`: To iterate over a list of selected elements.
- `filter()`: Select elements based on CSS selectors.
- `text()`: Extract the text content of the selected elements.

## Parsing HTML in PHP: Comparison Table

You can compare the three approaches to parsing HTML in PHP explored here in the summary table below:

|     | **\\DOM\\HTMLDocument** | **Simple HTML DOM Parser** | **Symfony’s DomCrawler** |
| --- | --- | --- | --- |
| **Type** | Native PHP component | External Library | Symfony Component |
| **GitHub Stars** | —   | 880+ | 4,000+ |
| **XPath Support** | ❌   | ✔️  | ✔️  |
| **CSS Selector Support** | ❌   | ✔️  | ✔️  |
| **Learning Curve** | Low | Low to Medium | Medium |
| **Simplicity of Use** | Medium | High | High |
| **API** | Basic | Rich | Rich |

## Conclusion

While these solutions work, they won’t be effective if the target web pages rely on JavaScript for rendering. In such cases, simple HTML parsing approaches like those above won’t suffice. Instead, you'll need a fully-featured scraping browser with advanced HTML parsing capabilities, such as [Scraping Browser](https://brightdata.com/products/scraping-browser).

If you want to bypass HTML parsing and access structured data instantly, explore our [ready-to-use datasets](https://brightdata.com/products/datasets), covering hundreds of websites!

Create a Bright Data account today and start testing our data and scraping solutions with a free trial! 
