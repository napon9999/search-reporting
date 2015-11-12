# What's Included? #

  1. `readme.txt` -- this file.
  1. `asr.py` -- this is a web server that will run on a custom port and log the clicks. This will always be running.
  1. `asr.js` -- this is the javascript file that the XSLT pulls in.
  1. `report.py` -- this can be run to generate HTML reports from the CSV logs.  It can be run from a cron job.  _Note: this should be run from a web server dir so the CSV log files are visible._
  1. `blue.gif` -- this is a basic graph to create report graphs.

# Overview #

You will need:
  1. A web server (e.g. Apache) that can feed out:
    * the CSV logs
    * the generated reports
    * the JS
  1. Python (can download from [python.org](http://python.org))

# Getting Started #
## 1. Place all 4 of the above listed files on a web server that has python. ##
  * We will refer to this web server below as YOUR\_HOST.
  * The scripts will be creating your log files in this directory on this drive so make sure there is enough room, the server is fast enough, it is not already overloaded with other processes, etc.

## 2. Add the Javascript variables to your front end ##

See below for what should be added:
```
<!-- **********************************************************************
 Search results (do not customize)
     ********************************************************************** -->
<xsl:template name="search_results">
<html>

  <!-- *** HTML header and style *** -->
  <xsl:call-template name="langHeadStart"/>
    <xsl:call-template name="redirect_if_few_results"/>
    <title><xsl:value-of select="$result_page_title"/>:
      <xsl:value-of select="$space_normalized_query"/>
    </title>
    <xsl:call-template name="style"/>
    <script type="text/javascript">
      <xsl:comment>
        function resetForms() {
          for (var i = 0; i &lt; document.forms.length; i++ ) {
              document.forms[i].reset();
          }
        }
// ---------ADD THE FOLLOWING LINES--------------

        // Search query
        var page_query = &quot;<xsl:value-of select="$stripped_search_query"/>&quot;
        // Starting page offset, usually 0 for 1st page, 10 for 2nd, 20 for 3rd.
        var page_start = &quot;<xsl:value-of select="/GSP/PARAM[@name='start']/@value"/>&quot;

// -----------------------------------------------

      //</xsl:comment>
    </script>
  <xsl:call-template name="langHeadEnd"/>
```

## 3. Pull in the ASR Javascript file that will do the actual logging ##

In the same
```
<!-- **********************************************************************
 Search results (do not customize)
     ********************************************************************** -->
```

section, right before:

```
  <!-- *** HTML footer *** -->
  </body>
</html>
```

add the following lines:
```
 <script language='javascript' src='http://YOUR_HOST/asr.js'></script>
 <script type='text/javascript'>
   clk(null, page_query, 'load', '', '', page_start);
 </script>

```

Be sure to change YOUR\_HOST above to the host and location where the javascript file can be found.

## 4. Add hooks to the search results to not only log the click but also the rank of the search result. ##

In this section:
```

<!-- **********************************************************************
  A single result (do not customize)
     ********************************************************************** -->
```

replace the following line:

```
      <xsl:text disable-output-escaping='yes'>&lt;a href="</xsl:text>
```

with

```
      <xsl:text disable-output-escaping='yes'>&lt;a
            ctype="c"
      </xsl:text>
            rank=&quot;<xsl:value-of select="position()"/>&quot;
      <xsl:text disable-output-escaping='yes'>
            href="</xsl:text>
```

## 5. Add hooks the rest of the 'a' tags that have an href (i.e. links) and add a ctype attribute. ##

In other words, you want to tag all the different types of links that users might click so that you can report on them easily.  If you miss a link, it's not the end of the world.  They will be automatically converted to ctype=OTHER.  However, the more places where you add ctype attributes, the richer your reporting will be and the better analysis you'll be able to perform (like, 30% of all clicks are Key Matches!).

For instance, to find out how many times users are clicking on the 'Next page' link, change:
```
 <a href="search?{$search_url}&amp;start={$view_begin + $num_results - 1}">
```

to:

```
<a ctype="nav.next" href="search?{$search_url}&amp;start={$view_begin + $num_results - 1}">
```

## 6. Edit asr.js ##
In your `asr.js` file, replace YOUR\_HOST with your host name where your python script can be found in the line below.

```
    var src = "http://YOUR_HOST:8879/click?" + 
```

## 7. Edit report.py ##
In your `report.py` file, replace YOUR\_GSA with the full hostname of your GSA (e.g. search.mycompany.com) and add any other GSA parameters after the `q=` parameter as you might like.  This is because there is a part of the report that allows you to showcase the top queries and this will allow viewers of the report to be able to click on them and see the results that are returned very easily.

Also, in the same file, replace YOUR\_HOST with the full hostname where you have placed your javascript & gif files.  This is so your html report graphs are able to find the necessary images.

## 8. Run asr.py ##
This will start a web server on port 8879.  Here's how the click logging will work:
  1. A user will make a query or click a link on the GSA
  1. The GSA's HTML code (thanks to the hooks you added to the XSLT above) now contains onclick events that will send both queries and clicked links to this web server (script) on port 8879.
  1. The asr.py script will log the event (.txt file) and append a line to your .csv file.
  1. The web server on port 8879 will then send a response back to the HTML.
  1. You can keep this process running as long as you want.  A new log file and csv file will automatically be created each day (as long as there are at least some clicks each day).

## 9. Run report.py ##
At any point, you can make copies of the .csv files, parse the data, and run any analysis you'd like using your favorite Business Intelligence tool.  The logging is very thorough and detailed and includes every query loaded and every subsequent click by time done, by IP address, etc.  For more information on the fields, see the asr.py file.

At any point, you can run a sample HTML-generating reporting tool (Pythong script) we created for you called report.py.  When run, this script will look in the current directory for any .csv files and create respective .html files.  You can then view these reports to see information such as how many queries occurred without any clicks (opportunity queries), how many satisfied queries occurred, and important metrics such as average click rank.  An average click rank of 8.5, for instance, is not very good and implies that, on average, users are having to go to the bottom of the search results to find the best result.  An average click rank of 1.5, however, is extremely good and implies that half the users click on the first search result and half on the second.  Over time, with proper search quality optimization, you should see your average click rank get lower.  This can be accomplished by implementing some of the best practices listed in the [Understanding the Search Experience](http://code.google.com/apis/searchappliance/documentation/50/admin_searchexp/ce_understanding.html) published document.

Good luck!