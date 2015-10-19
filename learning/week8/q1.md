# Q1

## What is the average rating per city in some chosen state of open businesses that have at least some chosen rating?

<div style="border:1px grey solid; padding:5px;">
    <div><h5>Open?</h5>
        <input id="arg1" type="text" value="yes/no"/>
    </div>
    <div><h5>State</h5>
        <input id="arg2" type="text" value="AZ"/>
    </div>
    <div><h5>Rating</h5>
        <input id="arg3" type="text" value="3.5"/>
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

console.log('lodash version:', _.VERSION)

$.get('http://bigdatahci2015.github.io/data/yelp/yelp_academic_dataset_business.5000.json.lines.txt')
    .success(function(data){        
        var lines = data.trim().split('\n')

        // convert text lines to json arrays and save them in `items`
        items = _.map(lines, JSON.parse)

        console.log('number of items loaded:', items.length)

        console.log('first item', items[0])

        $('.myviz').html('Data Loaded')
     })
     .error(function(e){
         console.error(e)
     })

function viz(arg1, arg2, arg3){    

    // define a template string
    var tplString = '<g transform="translate(0 ${d.y})"> \
                    <text x="100" y="20">${d.label}</text> \
                    <rect x="30"   \
                         width="${d.width}" \
                         height="20"    \
                         style="fill:${d.color};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    </g>'

    // compile the string to get a template function
    var template = _.template(tplString)

    function computeX(d, i) {
        return 0
    }

    function computeWidth(d, i) {        
        return d[1] * 10;
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i) {
        return 'red'
    }

    function computeLabel(d, i) {
        return d[0] + ', ' + d[1];
    }

    items = _.chain(items)
                .filter(function(obj) { return (obj.open == arg1);
                }).filter(function(obj) { return (obj.state == arg2);
                }).filter(function(obj) { return (obj.stars >= arg3);
                }).groupBy(function(obj) { return obj.city;
                }).mapValues(function(arr) {
                    return _.reduce(arr, function(p, n) {
                        return (p + n.stars);
                    }, 0) / arr.length;
                }).pairs().value()

    var viz = _.map(items, function(d, i){
                return {
                    x: computeX(d, i),
                    y: computeY(d, i),
                    width: computeWidth(d, i),
                    color: computeColor(d, i),
                    label: computeLabel(d, i)
                }
             })
    console.log('viz', viz)

    var result = _.map(viz, function(d){
             // invoke the compiled template function on each viz data
             return template({d: d})
         })
    console.log('result', result)

    $('.myviz').html('<svg width="100%" height="100%">' + result + '</svg>')
}

$('button#viz').click(function(){    
    var arg1 = ($('input#arg1').val() == 'yes')?true:false;
    var arg2 = $('input#arg2').val();
    var arg3 = parseFloat($('input#arg3').val());
    viz(arg1, arg2, arg3)
})

{% endscript %}
