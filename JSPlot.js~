/**
 * User: Austen
 * Date: 7/23/13
 * Time: 3:21 PM
 */
var Plotter = function(canvasId,width,height){
    if(!width)
        width = 800
    if(!height)
        height = 600;

    this.WIDTH = width;
    this.HEIGHT = height;
    /* SETTINGS */
    this.PADDING = 25;
    this.stream_max = 100;

    this.canvas = $("#"+canvasId);
    this.canvas.attr("width",(this.WIDTH+this.PADDING)+"px");
    this.canvas.attr("height",(this.HEIGHT+this.PADDING)+"px");
    this.canvas.css({"border":"thin black solid","margin": "10px"});
    this.renderer = document.getElementById(canvasId).getContext("2d");

    /* Attach tool-tip event to mouse move */
    var self = this;
    $("body").mousemove(function(e){self.displayTooltip(e)});
    $("body").click(function(e){self.displayTooltip(e,e.button)});
    this.canvas.css({"cursor":"pointer"});

    /* Current graph data--filled in by a call to graphData() */
    this.stream_Outputs = {};
    this.curr_Inputs = [];
    this.curr_Outputs = [];
    this.curr_x_Bounds = {"max":0,"min":0,"step":1};
    this.curr_y_Bounds = {"max":0,"min":0,"step":1};
}

Plotter.prototype.constructor = Plotter;

/* Graphs a list of inputs vs sets of output lists
 * @param inputs A list of inputs
 * @param outputs A list or a set of lists of corresponding outputs
 * @param input_label
 * @param output_labels A list or a set of lists of strings describing outputs
 * @param x_bounds, y_bounds A key/val object {max:<#>,min:<#>}
 */
Plotter.prototype.graphData = function(inputs,outputs,input_label,output_labels,x_bounds,y_bounds){
    /* Ensure inputs and outputs are set */
    if(!inputs && !outputs){
        inputs = this.curr_Inputs;
        outputs = this.curr_Outputs;
    }
    if(!outputs[0].length)
        outputs = [outputs];
    if(inputs=="time"){
        inputs = this.createRange(0,outputs.map(function(a){return a.length}).reduce(function(a,b){return Math.max(a,b)}));
        input_label = "T";
    }

    /* Ensure labels are set */
    if(!input_label){
        if(!this.curr_Input_Label){
            input_label = "X";
        }else{
            input_label = this.curr_Input_Label;
        }
    }
    if(!output_labels){
        if(!this.curr_Output_Labels){
            output_labels = [];
            outputs.forEach(function(e,i,a){output_labels.push("X_"+i)});
        }else{
            output_labels = this.curr_Output_Labels;
        }
    }
    if(!output_labels[0].length)
        output_labels = [output_labels];

    /* Ensure bounds are set */
    if(!x_bounds){
        x_bounds = this.getBounds(inputs);
    }
    if(!y_bounds){
        y_bounds = this.getBounds(outputs);
    }
    /* Update current graph data */
    this.curr_Inputs = inputs;
    this.curr_Outputs = outputs;
    this.curr_Input_Label = input_label;
    this.curr_Output_Labels = output_labels;
    this.curr_x_Bounds = x_bounds;
    this.curr_y_Bounds = y_bounds;

    this.clearGraph();
    this.setup_Axes(x_bounds,y_bounds,10);

    var point_size = 3;
    for(var j=0;j<outputs.length;j++){
        var currOutput = outputs[j];
        var lineColorStr = "rgba("+Math.floor(Math.abs(Math.sin(j/Math.PI))*250)+","+Math.floor(Math.random()*120)+","+Math.floor(Math.abs(Math.cos((j+1)/Math.PI))*200)+",1.0)";
        var innerColorStr = "rgba("+Math.floor(Math.random()*100+150)+","+Math.floor(Math.random()*100+150)+","+Math.floor(Math.random()*100+150)+",1)";

        var prev_scr;
        for(var i=0;i<currOutput.length;i++){
            //console.log(inputs[i]+","+currOutput[i]+"-->"+x+","+y);
            var scr = this.convertGraphCoords([inputs[i],currOutput[i]]); // get screen coordinates from input point
            if(i>0){
                this.renderer.save();
                this.renderer.lineWidth = 1;
                this.renderer.strokeStyle = lineColorStr;
                //this.renderer.globalCompositeOperation = "darken";
                this.renderer.beginPath();
                this.renderer.moveTo(prev_scr[0],prev_scr[1])
                this.renderer.lineTo(scr[0],scr[1]);
                this.renderer.stroke();
                this.renderer.restore();
            }
            this.renderer.fillStyle = lineColorStr;
            this.renderer.fillRect(scr[0]-point_size/2,scr[1]-point_size/2,point_size,point_size);
            prev_scr = scr;
        }
    }
}

Plotter.prototype.graphStream = function(streamName,newData){
    var stream_max = this.stream_max;
    var streams = this.stream_Outputs;
    if(!streams.hasOwnProperty(streamName)){
        streams[streamName] = [newData];
    }else{
        streams[streamName].push(newData);
    }

    var outputs = [];
    Object.keys(streams).forEach(function(e){
        if(streams[e].length>stream_max)
            streams[e].shift();
        outputs.push(streams[e])
    });
    var labels = Object.keys(streams);
    this.graphData("time",outputs,"time",labels);
}

Plotter.prototype.updateBounds = function(bounds){
    this.curr_x_Bounds["max"] = (!("x_max" in bounds))?this.curr_x_Bounds["max"]:bounds["x_max"];
    this.curr_x_Bounds["min"] = (!("x_min" in bounds))?this.curr_x_Bounds["min"]:bounds["x_min"];
    this.curr_y_Bounds["max"] = (!("y_max" in bounds))?this.curr_y_Bounds["max"]:bounds["y_max"];
    this.curr_y_Bounds["min"] = (!("y_min" in bounds))?this.curr_y_Bounds["min"]:bounds["y_min"];
    this.graphData(null,null,null,null,this.curr_x_Bounds,this.curr_y_Bounds);
}

Plotter.prototype.setup_Axes = function(x_bounds,y_bounds,num_steps){
    this.renderer.fillStyle = "rgb(0,0,0)"
    this.renderer.strokeStyle = "rgba(0,0,0,1)";
    this.renderer.lineWidth = 1;

    /* Draw X-Axis */
    var start_y = this.HEIGHT;
    var zero_y = (this.HEIGHT-this.PADDING)-(this.HEIGHT-this.PADDING*2)*(0-y_bounds.min)/(y_bounds.max-y_bounds.min);
    this.renderer.beginPath();
    this.renderer.moveTo(0,zero_y);
    this.renderer.lineTo(this.WIDTH+this.PADDING,zero_y);
    this.renderer.stroke();

    /* Draw Y-Axis */
    var start_x = this.PADDING;
    var zero_x = (this.PADDING)+(this.WIDTH-this.PADDING*2)*(0-x_bounds.min)/(x_bounds.max-x_bounds.min);
    this.renderer.beginPath();
    this.renderer.moveTo(zero_x,0);
    this.renderer.lineTo(zero_x,this.HEIGHT); // Y axis
    this.renderer.stroke();

    /* Draw tick lines */
    this.renderer.strokeStyle = "rgba(0,0,0,0.4)";
    this.renderer.font = "10pt calibri";
    this.renderer.lineWidth = 0.5;

    var point_size = 5;
    var step_size_pos = Math.ceil((x_bounds.max)/num_steps);
    var step_size_neg = x_bounds.min==0?1:Math.ceil(Math.abs((x_bounds.min)/num_steps));
    var step_size = Math.max(step_size_pos,step_size_neg);
    this.curr_x_Bounds['step'] = step_size;
    for(var i=0,j=0;i<=x_bounds.max || j>=x_bounds.min;i+=step_size,j-=step_size){
        var x_pos = (this.PADDING)+(this.WIDTH-this.PADDING*2)*(i-x_bounds.min)/(x_bounds.max-x_bounds.min);
        var x_neg = (this.PADDING)+(this.WIDTH-this.PADDING*2)*(j-x_bounds.min)/(x_bounds.max-x_bounds.min);


        this.renderer.beginPath();
        if(i<=x_bounds.max && i!=0){
            this.renderer.fillText((i).toFixed(2),x_pos,zero_y+10)
            this.renderer.moveTo(x_pos,0);
            this.renderer.lineTo(x_pos,this.HEIGHT);
        }
        if(j>=x_bounds.min && j!=0){
            this.renderer.fillText((j).toFixed(2),x_neg,zero_y+10)
            this.renderer.moveTo(x_neg,0);
            this.renderer.lineTo(x_neg,this.HEIGHT);
        }
        this.renderer.stroke();
    }

    /* Positive y-ticks */
    var step_size_pos = Math.ceil((y_bounds.max)/num_steps);
    var step_size_neg = y_bounds.min==0?1:Math.ceil(Math.abs((y_bounds.min)/num_steps));
    var step_size = Math.max(step_size_pos,step_size_neg);
    this.curr_y_Bounds['step'] = step_size;
    this.renderer.save()
    this.renderer.fillStyle = "rgba(0,0,0,0.6)";
    for(var i=0,j=0;i<=y_bounds.max || j>=y_bounds.min;i+=step_size,j-=step_size){
        var y_pos = (this.HEIGHT-this.PADDING)-(this.HEIGHT-this.PADDING*2)*(i-y_bounds.min)/(y_bounds.max-y_bounds.min);
        var y_neg = (this.HEIGHT-this.PADDING)-(this.HEIGHT-this.PADDING*2)*(j-y_bounds.min)/(y_bounds.max-y_bounds.min);

        this.renderer.beginPath();
        if(i<=y_bounds.max && i!=0){
            this.renderer.fillText((i).toFixed(2),this.WIDTH-this.PADDING,y_pos)
            this.renderer.moveTo(0,y_pos);
            this.renderer.lineTo(this.WIDTH+this.PADDING,y_pos);
        }
        if(j>=y_bounds.min && j!=0){
            this.renderer.fillText((j).toFixed(2),this.WIDTH-this.PADDING,y_neg)
            this.renderer.moveTo(0,y_neg);
            this.renderer.lineTo(this.WIDTH+this.PADDING,y_neg);
        }
        this.renderer.stroke();
        //console.log(i);
    }
    this.renderer.restore();
}

Plotter.prototype.clearGraph = function(){
    this.renderer.save();
    this.renderer.fillStyle = "#FFF";
    this.renderer.fillRect(0,0,this.canvas.width(),this.canvas.height());
    this.renderer.restore();
}

/* Naive max finder... todo: possibly just sort the data using heap sort or something */
/* @param Takes any number of arrays to find the bounds between them all */
Plotter.prototype.getBounds = function(list){
    if(!list[0].length)
        list = [list];
    var currMax = 0;
    var currMin = 0;
    for(var j=0;j<list.length;j++){
        var currList = list[j];
        for(var i=0;i<currList.length;i++){
            if(currList[i]>currMax){
                currMax = currList[i];
            }

            if(currList[i]<currMin){
                currMin = currList[i];
            }
        }
    }
    if(currMax==0 && currMin==0){
        currMax = 1;
    }
    return {"min": currMin, "max": currMax};
}

/* Converts screen coords to graph coords */
Plotter.prototype.convertScreenCoords = function(screenCoords){
    var scrX = screenCoords[0];
    var scrY = screenCoords[1];
    var x = (scrX-this.PADDING)*(this.curr_x_Bounds.max-this.curr_x_Bounds.min)/(this.WIDTH-2*this.PADDING) + this.curr_x_Bounds.min;
    var y = (this.HEIGHT-scrY-this.PADDING)*(this.curr_y_Bounds.max-this.curr_y_Bounds.min)/(this.HEIGHT-2*this.PADDING) + this.curr_y_Bounds.min;
    return [x,y];
}

/* Converts graph coords to screen coords */
Plotter.prototype.convertGraphCoords = function(graphCoords){
    var graphX = graphCoords[0],
        graphY = graphCoords[1];
    var x_bounds = this.curr_x_Bounds,
        y_bounds = this.curr_y_Bounds;
    var x = (this.PADDING)+(this.WIDTH-this.PADDING*2)*(graphX-x_bounds.min)/(x_bounds.max-x_bounds.min);
    var y = (this.HEIGHT-this.PADDING)-(this.HEIGHT-this.PADDING*2)*(graphY-y_bounds.min)/(y_bounds.max-y_bounds.min);
    return [x,y];
}

/* Creates a range list */
Plotter.prototype.createRange = function(start,end){
    if(start>end)
        throw new Error("Start parameter must be less than end parameter");
    var range = [];
    for(var i=start;i<end;i++){
        range.push(i);
    }
    return range;
}

Plotter.prototype.displayTooltip = function(event,button){
    var mX = event.pageX-this.canvas.offset().left;
    var mY = event.pageY-this.canvas.offset().top;

    var DIV_ID = "grapher_tooltip";
    if($("#"+DIV_ID).length==0){
        $("body").append("<div id='"+DIV_ID+"'></div>");
        $("#"+DIV_ID).css({"background":"rgba(250,200,50,0.8)","border":"thin solid black","position":"absolute"});
    }
    var $tooltip = $("#"+DIV_ID);
    /* toggle tooltip on click */
    if(button==0){
        if($tooltip.is(":visible")){
            $tooltip.hide();
        }else if(!$tooltip.is(":visible"))
            $tooltip.show();
    }

    if(mX<0 || mY<0 || mX > this.canvas.width() || mY > this.canvas.height()){
        if($tooltip.is(":visible"))
            $tooltip.hide();
    }else{
        /* determine if mouse is over a line */
        var graphCoords = this.convertScreenCoords([mX,mY]);
        var tol_y = this.curr_y_Bounds['step']/10;
        var tol_x = this.curr_x_Bounds['step']/10;
        var lineNum=[];
        var x = null,
            y = [];
        for(var j=0;j<this.curr_Inputs.length;j++){
            if(graphCoords[0]>=this.curr_Inputs[j]-tol_x && graphCoords[0]<this.curr_Inputs[j]+tol_x){
                x = j;
            }
        }
        for(var i=0;i<this.curr_Outputs.length;i++){
            var currOutput = this.curr_Outputs[i];
            if(graphCoords[1]>=(currOutput[x]-tol_y) && graphCoords[1]<=(currOutput[x]+tol_y)){
                lineNum.push(i);
                y.push(currOutput[x]);
            }
        }
        /* Write to tooltip and position over mouse */
        if(lineNum.length>0){
            $tooltip.html("");
            for(var i in lineNum){
                $tooltip.append(this.curr_Output_Labels[lineNum[i]]+": ("+[x,y[i]].join(",")+")<br>");
            }
        }else{
            $tooltip.html(graphCoords.map(function(v){return v.toFixed(2)}).join(",")+"<br>tol_x: "+tol_x+", tol_y: "+tol_y);
        }
        var left = event.pageX-$tooltip.width()/2;
        left = (left<=0)?event.pageX+5:left;
        var top = event.pageY+20;
        top = $tooltip.height()+(top-this.canvas.offset().top)>this.canvas.height()?event.pageY-$tooltip.height():top;
        $tooltip.css({"left":left,"top":top});
    }
}
