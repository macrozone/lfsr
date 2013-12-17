	
	if Meteor.isClient

		sampleConfig = 
			bits: [true,false,true,false]
			ops: ['not', 'id', 'xor']


		
		
		Session.set "setupBits", sampleConfig.bits
		Session.set "setupOps", sampleConfig.ops	

		sanitizeBits = (asString) ->
			_(asString.split ",").map (entry) -> entry != '0'

		sanitizeOps = (asString) ->
			asString.split ","

		
				

		Template.setup.setupBits = ->
			(_(Session.get "setupBits").map (bit) -> if bit then "1" else "0").join ","
		
		Template.setup.setupOps = ->
			(Session.get "setupOps").join ","

		onChangeBits = (event)->
				Session.set "setupBits", sanitizeBits $(event.target).val()
				reset()
		onChangeOps= (event)->
				Session.set "setupOps", sanitizeOps $(event.target).val()
				reset()


		Template.setup.events =
			"change .bits": onChangeBits
			"keypress .bits": _.debounce onChangeBits, 1200
			
			"change .ops": onChangeOps
			"keypress .ops": _.debounce onChangeOps, 1200
	
		
		lfsr = null
		chart = null
		runTimeout = null

		reset = ->
			Meteor.clearTimeout runTimeout
			lfsr = new LFSR 
				bits: Session.get "setupBits"
				ops: Session.get "setupOps"

			Session.set "lsfrState", lfsr
			Session.set "sequence", []
			if chart 
				chart.series[0].update data: []
				chart.series[1].update data: []

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

			checkSize = 64
			if sequence.length == checkSize
				fft = new FFT checkSize
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


		

			