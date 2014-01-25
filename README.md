# JSPlot

#### JSPlot is a simple javascript plotting library, useful for quick debugging or visualization of signals.

To use JSPlot, simply initialize the Plotter object onto a canvas element.
Plot data retrospectively using graphData, or on the fly with graphStream.

    <html>
    <head>
    <script src="JSPlot.js"></script>
    <script src="http://code.jquery.com/jquery-latest.min.js"
            type="text/javascript"></script>
    </head>
    <body>
    <canvas id='plot1'>	
    <script>
    plot1 = new Plotter('plot1')
    for(var i=0;i<100;i++){
            plot1.graphStream('hey',Math.sin(i*(2.0*Math.pi)/100.0))
    }
    </script>
    </canvas>
    </body>
    </html>

###### Note that I wrote this as a personal tool, and it has primarily only been tested with Chrome.
