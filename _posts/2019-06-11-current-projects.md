---

layout: post
title:  "current projects"
date:   2019-06-11 06:00:00 -0700
categories: [personal,tutorial]
tags: [apache,solr,js,css]

---

![image1](/assets/solr-logo.png "Jekyll")
##### **it's been awhile, let's talk**

---

So I've fallen off the radar for a bit, but it's been for good reasons.  I was able to make the transition from Systems Administrator to Programmer!  Goal 1 CHECK!
But the real question is now..  how do I go further?  Well, let us start by talking about some of the technologies I'm currently working with.



###### **Apache Solr**
In my first month on the job a challenge was presented to me.  Build a search application to replace a deprecated search app that is holding back a server migration.  Oh, no problem.  Let me introduce you to my friend Solr and his cousin Nutch.  These two **FREE** and **OPEN-SOURCE** programs(both carrying the Apache banner) became my best friends for two weeks.  In that time, I was able to slam a little jQuery together to produce a quick little search application, that not only utilized multiple Solr cores for relevant search. But also featured standard search characteristics like hit highlighting, dynamic paging, and lighting fast speeds.  Did I mention, these are all features Solr provides out of the box?  The best part is, with a little reading of the [documentation](https://www.manning.com/books/solr-in-action?a_bid=39472865&a_aid=1) it's not too hard to implement.

I won't be able to dive into all the code, but let's at least look at some key pieces and then later we'll dive into Nutch.

**Query Building**
```javascript
//rebuild query string depending on variables given
function getQuery(userInput,withHl,start){    

var query,
    baseUrl = "http://$SOLRSERVER:8983/solr/"
    core = getCookie("searchType"), //core to query  
    qt = "/select?", //request handler
    start = getPaging(start)
    paging = 'rows=' + rows + '&' + 'start=' + start,
    df = "df=" + "content" + "&", //default search field
    fl = "title%2Ccontent%2Curl%2Canchor&", //field list to return(comma separated) (%2C = "," escape)
    hl = "hl=on&",
    hlS = "hl.snippets=2&",
    hlPre = 'hl.simple.pre=<span class="highlight">&',
    hlPost = "hl.simple.post=</span>&",
    q = ["q=",userInput,"~2&"]; 
//build url again, depending on if paging & hit-high lighting are needed
(withHl == false ? query = baseUrl + core + qt + df + fl + q.join('') + paging :query = baseUrl + core + qt + hl + hlS + hlPre + hlPost + df + fl + q.join('') + paging);
return(query)
};
```
The above snippet is a look at how I built my queries for Solr.  This is the key to my app because not only do the queries to Solr provide search results, but they also tell Solr what features you'd like returned with those results.(hit-highlighting, paging, etc).  This is useful to know when building out the page markup.  I won't go into specifics about what the query string does/is, there are plenty of docs on the web regarding that.

**Paging**

_Credit to [kottenator](https://gist.github.com/kottenator) for [this](https://gist.github.com/kottenator/9d936eb3e4e3c3e02598) paging algorithm._
```javascript
//creates page numbers/previous/next buttons
function initPageNumbers(currentPage){ 
    //define empty l var
    var l;
    //clear markup if any exists
    $('#pageNums').html('');
    //get request to solr
    $.get(qStr, function(data){  
        //total number of results returned
        total_rows = data['response']['numFound'],
        //total num divided by rows to return
        last = Math.ceil(total_rows / rows),
        current = currentPage,
        delta = 2,
        left = current - delta,
        right = current + delta + 1,
        range = [],
        rangeWithDots = []; 
        $('#pageNums').append(
            '<li><a id="previous" href="#'+
            getPaging(current,"backward")+ 
            '" class="btn btn-default" onclick="getPage('+
            getPaging(current - 1,"backward")  + 
            '); initPageNumbers(' + getPaging(current,"backward") + 
            ')">'+ 
            "Previous" + '</a></li>'); 

        for (i = 1; i <= last; i++) {
            (i == 1 || i == last || i >= left && i < right ? range.push(i):null) 
        }
          
            for (i of range) {
                if (l) {
                    if (i - 1 === 2){ 
                    } else if (i - l !== 1) {
                        $('#pageNums').append(
                        '<li><a href="#" class="btn btn-default">' + 
                        "..." + '</a></li>');
                        rangeWithDots.push('...')
                    }
                }
                $('#pageNums').append('<li><a href="#'+ 
                (i) + '" class="btn btn-default" onclick="getPage('+ 
                (i - 1) + '); initPageNumbers(' + (i) + ')">'+ 
                (i) + '</a></li>');
                rangeWithDots.push(i);
                l = i;
                num = rangeWithDots[2]; 
            } 
            (current == 1 ?  $('#pageNums').append(
            '<li><a id="next" href="#'+ getPaging(current,"foreward") +
            '" class="btn btn-default" onclick="getPage('+ 1 + '); initPageNumbers(' +
             getPaging(current,"foreward") + ')">'+ 
            "Next" + '</a></li>'):

            $('#pageNums').append(
            '<li><a id="next" href="#' + getPaging(current,"foreward") +
            '" class="btn btn-default" onclick="getPage('+
            getPaging(current - 1,"foreward") + '); initPageNumbers(' +
            getPaging(current,"foreward") + ')">'+ 
            "Next" + '</a></li>'));
            
            check(range[range.length - 1],currentPage);
    });
}
```
This would be the 2nd main chunk of my code, the paging.  It's kind of chunky, but it gets the job done. The way I went about creating this was finding an algorithm [someone](https://gist.github.com/kottenator) had already created for the paging, figuring out how it worked, and then using that as a _"framework"_ for my function.  From there I was able to create another function to wipe and rebuild the next results page and recursively call initPageNumbers() to rebuild the paging.


So... Obviously you'd be better off using a framework like [Velocity](https://velocity.apache.org/) or [Flask](http://flask.pocoo.org/) along with a [language specific Solr library](https://wiki.apache.org/solr/IntegratingSolr), but for this specific application the approach worked great.  

**NOTE:**

The rest of the code is specific to the application so it won't be necessary to present it, but most of it's purpose was to preserve the search input and specify which core to query using [cookies](https://www.w3schools.com/js/js_cookies.asp)(see getQuery(), var core), activate event listeners, and add some flair(think Google) to the page. 

-Paul 
