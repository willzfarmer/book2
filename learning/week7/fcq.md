# FCQ

Practice what you just learned on the FCQ dataset.

## Menu

### Group By

<div style="border:1px grey solid; padding:5px;">
<button id="viz-college">Gorup By College</button>
</div>

<div style="border:1px grey solid; padding:5px;">
Field: <input id="group-by-attribute" type="text" value="CrsPBAColl"/>
<button id="viz-group-by-attribute">Group By</button>
</div>

## Viz

<div class="myviz" style="width:100%; height:500px; border: 1px black solid;">
Data is not loaded yet
</div>

{% script %}
// pokemonData is a global variable
fcqData = 'not loaded yet'

$.get('/data/fcq.clean.json')
 .success(function(data){
     console.log('data loaded', data)
     fcqData = data          
     $('.myviz').html('number of records load:' + data.length)
 })


function vizCollege(){

    // process fcqData

    var groups = _.groupBy(fcqData, function(d){
        return d['CrsPBAColl']
    })

    var dataArray = _.map(_.pairs(groups), function(p){
        return {name: p[0], count: p[1].length}
    })

    console.log('dataArray', dataArray)


    // define a template string
    var tplString = '<g transform="translate(0 ${d.y})"> \
                    <rect   \
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
        return d['count'] / 5
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i) {
        return 'red'
    }

    var viz = _.map(dataArray, function(d, i){
                return {
                    x: computeX(d, i),
                    y: computeY(d, i),
                    width: computeWidth(d, i),
                    color: computeColor(d, i)
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

$('button#viz-college').click(vizCollege)

// TODO: complete the code below
// 1. Each horizontal bar should have a label
// 2. Each horizontal bar should have a number indicating the count
function vizGroupByAttribute(attributeName){
    // process fcqData

    var groups = _.groupBy(fcqData, function(d){
        return d[attributeName]
    })

    var dataArray = _.map(_.pairs(groups), function(p){
        return {name: p[0], count: p[1].length}
    })

    console.log('dataArray', dataArray)


    // define a template string
    var tplString = '<g transform="translate(0 ${d.y})"> \
                    <rect   \
                         width="${d.width}" \
                         height="20"    \
                         style="fill:${d.color};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    <text transform="translate(0 15)"> \
                        ${d.label} \
                    </text> \
                    </g>'

    // compile the string to get a template function
    var template = _.template(tplString)

    function computeX(d, i) {
        return 0
    }

    function computeWidth(d, i) {
        return d['count'] / 10
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i) {
        return 'red'
    }

    function computeLabel(d, i) {
        return d['name'] + ' ' + d['count']
    }

    var viz = _.map(dataArray, function(d, i){
                return {
                    x: computeX(d, i),
                    y: computeY(d, i),
                    width: computeWidth(d, i),
                    label: computeLabel(d, i),
                    color: computeColor(d, i)
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

$('button#viz-group-by-attribute').click(function(){
    var attributeName = $('input#group-by-attribute').val();
    vizGroupByAttribute(attributeName)
})  

{% endscript %}
