# Pokemon

Create interactive visualizations for the Pokemon data. Complete the script
so that when a user clicks on each menu button, there will be a different
bar chart shown in the Viz block.

You will be reusing the code you wrote for generating SVG bar charts a couple weeks
ago. The main challenge is to figure out how to connect your visualization code
with the front-end code (event handlers ... etc).

## Menu

<button id="viz-horizontal">Attack (Horizontal Bars)</button>
<button id="viz-vertical">Attack (Vertical Bars)</button>
<button id="viz-attack-defense">Attack vs. Defense</button>
<button id="viz-speed-defense">Speed vs. Defense</button>
<button id="viz-horizontal-sorted">Attack (sorted from low to high)</button>
<button id="viz-horizontal-sorted-desc">Attack (sorted from high to low)</button>
<button id="viz-attack-speed">Attack (width) vs. Speed (color)</button>

## Viz

<div class="myviz" style="width:100%; height:500px; border: 1px black solid;">
Data is not loaded yet
</div>

{% script %}
// pokemonData is a global variable
pokemonData = 'not loaded yet'

$.get('/data/pokemon-small.json')
 .success(function(data){
     console.log('data loaded', data)
     pokemonData = data
     $('.myviz').html('number of records load:' + data.length)
 })


function vizAsHorizontalBars(){
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
        return d['Attack'];
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i) {
        return 'red'
    }

    var viz = _.map(pokemonData, function(d, i){
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

    $('.myviz').html('<svg>' + result + '</svg>')
}

function attackVertical(){
    var tplString = '<g transform="translate(${d.x} ${d.y})"> \
                    <rect   \
                         width="20" \
                         height="${d.height}"    \
                         width="${d.width}"    \
                         style="fill:${d.color};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    </g>'

    // compile the string to get a template function
    var template = _.template(tplString)

    function computeX(d, i) {
        return i * 25
    }

    function computeY(d, i) {
        return 0;
    }

    function computeWidth(d, i) {
        return 20
    }

    function computeHeight(d, i) {
        return d['Attack'];
    }

    function computeColor(d, i) {
        return 'red'
    }

    var viz = _.map(pokemonData, function(d, i){
                return {
                    x: computeX(d, i),
                    y: computeY(d, i),
                    width: computeWidth(d, i),
                    height: computeHeight(d, i),
                    color: computeColor(d, i)
                }
             })
    console.log('viz', viz)

    var result = _.map(viz, function(d){
             // invoke the compiled template function on each viz data
             return template({d: d})
         })
    console.log('result', result)

    $('.myviz').html('<svg>' + result + '</svg>')
}

function attackDefense(){
    var tplString = '<g transform="translate(${d.ax} ${d.ay})"> \
                    <rect   \
                         height="${d.aheight}"    \
                         width="${d.awidth}"    \
                         style="fill:${d.acolor};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    <text transform="translate(0 15)"> \
                        ${d.alabel} \
                    </text> \
                    </g>'
    tplString += '<g transform="translate(${d.dx} ${d.dy})"> \
                    <rect   \
                         height="${d.dheight}"    \
                         width="${d.dwidth}"    \
                         style="fill:${d.dcolor};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    <text transform="translate(0 15)"> \
                        ${d.dlabel} \
                    </text> \
                    </g>'


    // compile the string to get a template function
    var template = _.template(tplString)

    function computeX(d, i, attack) {
        if (attack) {
            return 0;
        } else {
            return 150;
        }
    }

    function computeY(d, i, attack) {
        return i * 20
    }

    function computeWidth(d, i, attack) {
        if (attack) {
            return d['Attack'];
        } else {
            return d['Defense'];
        }
    }

    function computeHeight(d, i, attack) {
        return 20
    }

    function computeColor(d, i, attack) {
        if (attack) { return 'red' } else { return 'blue' }
    }

    function computeLabel(d, i, attack) {
        return d['Name']
    }

    var viz = _.map(pokemonData, function(d, i){
                return {
                    ax: computeX(d, i, true),
                    ay: computeY(d, i, true),
                    awidth: computeWidth(d, i, true),
                    aheight: computeHeight(d, i, true),
                    acolor: computeColor(d, i, true),
                    alabel: computeLabel(d, i, true),
                    dx: computeX(d, i, false),
                    dy: computeY(d, i, false),
                    dwidth: computeWidth(d, i, false),
                    dheight: computeHeight(d, i, false),
                    dcolor: computeColor(d, i, false),
                    dlabel: computeLabel(d, i, false)
                }
             })
    console.log('viz', viz)

    var result = _.map(viz, function(d){
             // invoke the compiled template function on each viz data
             return template({d: d})
         })
    console.log('result', result)

    $('.myviz').html('<svg>' + result + '</svg>')
}

function speedDefense(){
    var tplString = '<g transform="translate(${d.ax} ${d.ay})"> \
                    <rect   \
                         height="${d.aheight}"    \
                         width="${d.awidth}"    \
                         style="fill:${d.acolor};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    <text transform="translate(0 15)"> \
                        ${d.alabel} \
                    </text> \
                    </g>'
    tplString += '<g transform="translate(${d.dx} ${d.dy})"> \
                    <rect   \
                         height="${d.dheight}"    \
                         width="${d.dwidth}"    \
                         style="fill:${d.dcolor};    \
                                stroke-width:3; \
                                stroke:rgb(0,0,0)" />   \
                    <text transform="translate(0 15)"> \
                        ${d.dlabel} \
                    </text> \
                    </g>'


    // compile the string to get a template function
    var template = _.template(tplString)

    function computeX(d, i, attack) {
        if (attack) {
            return 0;
        } else {
            return 150;
        }
    }

    function computeY(d, i, attack) {
        return i * 20
    }

    function computeWidth(d, i, attack) {
        if (attack) {
            return d['Speed'];
        } else {
            return d['Defense'];
        }
    }

    function computeHeight(d, i, attack) {
        return 20
    }

    function computeColor(d, i, attack) {
        if (attack) { return 'red' } else { return 'blue' }
    }

    function computeLabel(d, i, attack) {
        return d['Name']
    }

    var viz = _.map(pokemonData, function(d, i){
                return {
                    ax: computeX(d, i, true),
                    ay: computeY(d, i, true),
                    awidth: computeWidth(d, i, true),
                    aheight: computeHeight(d, i, true),
                    acolor: computeColor(d, i, true),
                    alabel: computeLabel(d, i, true),
                    dx: computeX(d, i, false),
                    dy: computeY(d, i, false),
                    dwidth: computeWidth(d, i, false),
                    dheight: computeHeight(d, i, false),
                    dcolor: computeColor(d, i, false),
                    dlabel: computeLabel(d, i, false)
                }
             })
    console.log('viz', viz)

    var result = _.map(viz, function(d){
             // invoke the compiled template function on each viz data
             return template({d: d})
         })
    console.log('result', result)

    $('.myviz').html('<svg>' + result + '</svg>')
}

function attack_sort_asc(){
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
        return d['Attack'];
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i) {
        return 'red'
    }

    function computeLabel(d, i) {
        return d['Name']
    }

    var viz = _.chain(pokemonData)
                .sortBy(function(obj) {
                    return obj['Attack'];
                }).map(function(d, i){
                    return {
                        x: computeX(d, i),
                        y: computeY(d, i),
                        label: computeLabel(d, i),
                        width: computeWidth(d, i),
                        color: computeColor(d, i)
                    }
                }).value()
    console.log('viz', viz)

    var result = _.map(viz, function(d){
             // invoke the compiled template function on each viz data
             return template({d: d})
         })
    console.log('result', result)

    $('.myviz').html('<svg>' + result + '</svg>')
}

function attack_sort_desc(){
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
        return d['Attack'];
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i) {
        return 'red'
    }

    function computeLabel(d, i) {
        return d['Name']
    }

    var viz = _.chain(pokemonData)
                .sortBy(function(obj) {
                    return -obj['Attack'];
                }).map(function(d, i){
                    return {
                        x: computeX(d, i),
                        y: computeY(d, i),
                        label: computeLabel(d, i),
                        width: computeWidth(d, i),
                        color: computeColor(d, i)
                    }
                }).value()
    console.log('viz', viz)

    var result = _.map(viz, function(d){
             // invoke the compiled template function on each viz data
             return template({d: d})
         })
    console.log('result', result)

    $('.myviz').html('<svg>' + result + '</svg>')
}

function attack_speed(){
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
        return d['Attack'];
    }

    function computeY(d, i) {
        return i * 20
    }

    function computeColor(d, i) {
        return 'rgb(' + d['Speed'] + ', 0, 0)'
    }

    function computeLabel(d, i) {
        return d['Name']
    }

    var viz = _.chain(pokemonData)
                .sortBy(function(obj) {
                    return obj['Attack'];
                }).map(function(d, i){
                    return {
                        x: computeX(d, i),
                        y: computeY(d, i),
                        label: computeLabel(d, i),
                        width: computeWidth(d, i),
                        color: computeColor(d, i)
                    }
                }).value()
    console.log('viz', viz)

    var result = _.map(viz, function(d){
             // invoke the compiled template function on each viz data
             return template({d: d})
         })
    console.log('result', result)

    $('.myviz').html('<svg>' + result + '</svg>')
}

$('button#viz-horizontal').click(vizAsHorizontalBars)
$('button#viz-vertical').click(attackVertical);
$('button#viz-attack-defense').click(attackDefense);
$('button#viz-speed-defense').click(speedDefense);
$('button#viz-horizontal-sorted').click(attack_sort_asc);
$('button#viz-horizontal-sorted-desc').click(attack_sort_desc);
$('button#viz-attack-speed').click(attack_speed);

{% endscript %}
