	
	if Meteor.isClient


		config = 
			bits: [true,false,true,false,false,false,false]
			ops: ['not', 'id', 'xor','not','id','not']

			#bits: [true,false,false,false,false,false,false]
			#ops: ['not', 'id', 'xor','xor','id','not']
			#ops: ['id', 'id', 'id','id','id', 'id','id','id', 'id','id','id', 'id','id','id', 'id','id']


		LFSR = class 
			constructor: (config) ->
				config  = $.extend true, {}, config # clone the object
				@bits = config.bits
				@ops = config.ops


			step: =>
				out = @bits[0]
				for bit, index in @bits[1..]
					@bits[index] = @doOp @ops[index], bit, out	
				@bits[(@bits.length-1)] = out
				if out then 1 else 0

			doOp: (op, bit, overflowBit) ->
				switch op
					when 'id' then bit
					when 'not' then !bit
					when 'xor' then (!bit) != (!overflowBit)
		

		lfsr = {}
		chart = {}
		runTimeout = null

		reset = ->
		
			Meteor.clearTimeout runTimeout
			lfsr = new LFSR config
	
			Session.set "lsfrState", lfsr
			Session.set "sequence", []
	
		
		reset()
		
		step = ->
			Meteor.clearTimeout runTimeout
			lfsr.step()

		stepAndUpdateGui = ->
			Meteor.clearTimeout runTimeout
			out = step()
			sequence = Session.get "sequence"
			sequence = [] unless sequence?
			sequence.push out
			Session.set "sequence", sequence

			Session.set "lsfrState", lfsr

			fft = new FFT 64
			fftData = fft.forward sequence
	
			chart.series[0].update data: fftData.real.subarray 1
			chart.series[1].update data: fftData.imag.subarray 1

		run = ->
			
			stepAndUpdateGui()
			runTimeout = Meteor.setTimeout run, 1


		Template.chart.rendered = ->
			
			$chart = $(this.find(".chart")).highcharts
				chart:
					height: 600
					zoomType: 'x'
				legend:
					layout: "vertical"
					itemStyle:
						paddingTop: "8px"
						paddingBottom: "8px"
				type: "bar"
				title: "Periods"
			
				series: [
                	
               			(name: "real", data:[])
                		(name: "imag", data:[])
                		
                		]
			chart = $chart.highcharts()

		Template.lfsr.lfsr = ->
			Session.get "lsfrState"

		Template.lfsr.sequence = ->
			Session.get("sequence")?.reverse()

		Template.controls.events =
			"click .step": stepAndUpdateGui
			"click .run": run
			"click .reset": reset


		