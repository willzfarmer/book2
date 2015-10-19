# UI

## What businesses have category (V) in city (W) with at least (X) reviews and at least a rating of (Y) and are open on Day (Z)?

<div style="border:1px grey solid; padding:5px;">
    <div><h5>Category</h5>
        <input id="arg0" type="text" value="Restaurants"/>
    </div>
    <div><h5>City</h5>
        <input id="arg1" type="text" value="Phoenix"/>
    </div>
    <div><h5>Minimum Number of Reviews</h5>
        <input id="arg2" type="text" value="200"/>
    </div>
    <div><h5>Minimum Rating (0.0 - 5.0)</h5>
        <input id="arg3" type="text" value="4.0"/>
    </div>    
    <div><h5>Sort Option (None, Ascending, Descending))</h5>
        <input id="arg4" type="text" value="None"/>
    </div>    
    <div><h5>Day of Week (e.g., Wednesday)</h5>
        <input id="arg5" type="text" value="Wednesday"/>
    </div>    
    <div style="margin:20px;">
        <button id="viz">Vizualize</button>
    </div>
</div>

<div class="myviz" style="width:100%; height:500px; border: 1px black solid; padding: 5px;">
Data is not loaded yet
</div>

{% script %}
items = 'not loaded yet'

$.get('http://bigdatahci2015.github.io/data/yelp/yelp_academic_dataset_business.5000.json.lines.txt')
    .success(function(data){        
        var lines = data.trim().split('\n')

        // Convert text lines to json arrays and save them in `items`
        items = _.map(lines, JSON.parse)

        console.log('number of items loaded:', items.length)

        // Show in the myviz that the data is loaded
        $('.myviz').html('number of records load:' + data.length)

        console.log('first item', items[0])
     })
     .error(function(e){
         console.error(e)
     })

function viz(arg0, arg1, arg2, arg3, arg4, arg5) {    

    // Define a template string
    var tplString = '<g transform="translate(0 ${d.y})"> \
                    <text y="15">${d.label}</text> \
                    <rect x="310"   \
                         width="${d.width}" \
                         height="20"    \
                         style="fill:${d.color};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    <text x=315 y="15">${d.label2}</text> \
                    <text x=625 y="15">${d.label3}</text> \
                    </g>'

    // Compile the string to get a template function
    var template = _.template(tplString)

    function computeX(d, i) {
        return 0
    }

    function computeWidth(d, i) {        
        return d.stars * 60
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i, max_reviews) {
        if (d.review_count == max_reviews) {
            return 'yellow'
        } else {
            return 'red'
        }
    }

    function computeLabel(d, i) {
        return d.name + ' (' + d.review_count + ')'
    }

    function computeLabel2(d, i) {
        var string = ""
        var num_stars = _.floor(d.stars)
        for (i = 0; i < num_stars; i++) { string += '&#9733' }   // star solid [&#9733]
        if ((d.stars - num_stars) != 0) { string += '&#189' }    // fraction 1/2 [&#189]
        return ' Rating: ' + string
    }

    function computeLabel3(d, i) {
        var day, open, close
        if (arg5 == 'Sunday') { day = 'Sun' 
            if ( _.isUndefined(d.hours.Sunday) ) { open = ''; close = 'CLOSED' }
            else { open = d.hours.Sunday.open; close = d.hours.Sunday.close }
        }
        else if (arg5 == 'Monday') { day = 'Mon' 
            if ( _.isUndefined(d.hours.Monday) ) { open = ''; close = 'CLOSED' }
            else { open = d.hours.Monday.open; close = d.hours.Monday.close }
        }
        else if (arg5 == 'Tuesday') { day = 'Tue' 
            if ( _.isUndefined(d.hours.Tuesday) ) { open = ''; close = 'CLOSED' }
            else { open = d.hours.Tuesday.open; close = d.hours.Tuesday.close }
        }
        else if (arg5 == 'Wednesday') { day = 'Wed' 
            if ( _.isUndefined(d.hours.Wednesday) ) { open = ''; close = 'CLOSED' }
            else { open = d.hours.Wednesday.open; close = d.hours.Wednesday.close }
        }
        else if (arg5 == 'Thursday') { day = 'Thu' 
            if ( _.isUndefined(d.hours.Thursday) ) { open = ''; close = 'CLOSED' }
            else { open = d.hours.Thursday.open; close = d.hours.Thursday.close }
        }
        else if (arg5 == 'Friday') { day = 'Fri' 
            if ( _.isUndefined(d.hours.Friday) ) { open = ''; close = 'CLOSED' }
            else { open = d.hours.Friday.open; close = d.hours.Friday.close }
        }
        else if (arg5 == 'Saturday') { day = 'Sat' 
            if ( _.isUndefined(d.hours.Saturday) ) { open = ''; close = 'CLOSED' }
            else { open = d.hours.Saturday.open; close = d.hours.Saturday.close }
        }
        return day + ': ' + open + '-' + close
    }

    // UI processing logic

    console.log(arg0); console.log(arg1); console.log(arg2); 
    console.log(arg3); console.log(arg4); console.log(arg5);

    var filter_city = _.filter(items, function(d) {
        return (d.city == arg1)
    })
    var filter_reviews = _.filter(filter_city, function(d) {
        return (d.review_count >= parseInt(arg2))
    })
    var filter_stars = _.filter(filter_reviews, function(d) {
        return (d.stars >= parseFloat(arg3))
    })

    // Have filtered 'items' based on city, #reviews and rating (stars)
    // Find all objects with a category element matching user input 'arg0'

    var filter_categories = _.filter(filter_stars, function(f) {
        return _.some(f.categories, function(d) {
            return d == arg0
        })
    })
    var vizData = filter_categories

    // Determine if need to sort

    var sort = 0
    if (arg4 == "Ascending" || arg4 == "ascending") { sort = 1 }
    if (arg4 == "Descending" || arg4 == "descending") { sort = -1 }

    if (sort) {
        var vizData = _.sortBy(filter_categories, function(d) {
            return sort * d.stars
        })
    }

    // Take the first 20 items to visualize    
    var vizItems = _.take(vizData, 20)

    // Of the businesses with the highest rating (stars) highlight
    // the business with the greatest number of reviews

    var max_rating = _.max(_.pluck(vizItems, 'stars'))
    var filter_max = _.filter(vizItems, function(d) {
        return d.stars == max_rating
    })
    var max_reviews = _.max(_.pluck(filter_max, 'review_count'))

    var viz = _.map(vizItems, function(d, i) {                
        return {
            x: computeX(d, i),
            y: computeY(d, i),
            width: computeWidth(d, i),
            color: computeColor(d, i, max_reviews),
            label: computeLabel(d, i),
            label2: computeLabel2(d, i),
            label3: computeLabel3(d, i)
        }
    })

    var result = _.map(viz, function(d){
        return template({d: d}) // invoke the compiled template function on each viz data
    })

    $('.myviz').html('<svg width="100%" height="100%">' + result + '</svg>')
}

$('button#viz').click(function(){    
    var arg0 = $('input#arg0').val()
    var arg1 = $('input#arg1').val()
    var arg2 = $('input#arg2').val()
    var arg3 = $('input#arg3').val()
    var arg4 = $('input#arg4').val()
    var arg5 = $('input#arg5').val()
    viz(arg0, arg1, arg2, arg3, arg4, arg5)
})  

{% endscript %}

# Authors

This UI is developed by
* [William Farmer](http://github.com/willzfarmer)
* [Kevin Gifford](http://github.com/kevinkgifford)
* [Parker Illig](http://github.com/pail4944)
* [Robert Kendl](http://github.com/DomoYeti)
* [Andrew Krodinger](http://github.com/drewdinger)
* [John Raesly](http://github.com/jraesly)


