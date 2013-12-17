
	@setupExperiment = (config) ->

		lfsr = null
		chart = null
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