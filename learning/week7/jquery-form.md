# Input

<input id="show" type="text" value="hello"/> <button id="show">Show text</button>

<input id="setcolor" type="text" value="green"/> <button id="setcolor">Set Background Color</button>

<input id="setheight" type="text" value="200"/> <button id="setheight">Set Height</button>

## Bars (1)

<div style="border:1px grey solid; padding:5px;">
Number: <input id="bars1-number" type="text" value="5"/>
<button id="bars1">Show Bars (1)</button>
</div>

## Bars (2)

<div style="border:1px grey solid; padding:5px;">
Number: <input id="bars2-number" type="text" value="5"/>
Color:  <input id="bars2-color" type="text" value="red"/>
<button id="bars2">Show Bars (2)</button>
</div>

## Bars (3)

<div style="border:1px grey solid; padding:5px;">
Number: <input id="bars3-number" type="text" value="5"/>
Color:  <input id="bars3-color" type="text" value="red"/>
Height:  <input id="bars3-height" type="text" value="50"/>
<button id="bars3">Show Bars (3)</button>
</div>


## Viz

<div class="myviz" style="width:100%; height:100px; border: 1px black solid;">
</div>


{% script %}

$('button#show').click(function(){    
    var value = $('input#show').val()    
    console.log(value)
    $('.myviz').html(value)
})

$('button#setcolor').click(function(){
    $('.myviz').css('background-color',$('input#setcolor').val())
})

$('button#setheight').click(function(){
    $('.myviz').height($('input#setheight').val())
})


$('button#bars1').click(function(){
    var svg = "<svg>"
    for (i=0; i < $('input#bars1-number').val(); i++) {
        svg += "<rect height='50' width='10' x='" + i * 20 + "'/>"
    }
    svg += "</svg>"
    $('.myviz').html(svg)
})

$('button#bars2').click(function(){    
    var svg = "<svg>"
    for (i=0; i < $('input#bars2-number').val(); i++) {
        svg += "<rect height='50' width='10' x='" + i * 20 + "' style='fill:" + $('input#bars2-color').val() + "'/>"
    }
    svg += "</svg>"
    $('.myviz').html(svg)
})

$('button#bars3').click(function(){    
    var svg = "<svg>"
    for (i=0; i < $('input#bars3-number').val(); i++) {
        svg += "<rect height='" + $('input#bars3-height').val()
        svg += "' width='10' x='" + i * 20
        svg += "' style='fill:" + $('input#bars3-color').val() + "'/>"
    }
    svg += "</svg>"
    $('.myviz').html(svg)
})

{% endscript %}
