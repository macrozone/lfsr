	
	if Meteor.isClient


		config = 
			bits: [false,false,false,true]
			ops: ['not', 'id', 'xor']


		LFSR = class 
			constructor: (config) ->
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
		

		


		lfsr = new LFSR config
		Session.set "lsfrState", lfsr

		fft = new FFT 20
		
		runTimeout = null
		
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

			Session.set ""
			Session.set "lsfrState", lfsr

			console.log (fft.forward sequence).real

		run = ->
			
			stepAndUpdateGui()
			runTimeout = Meteor.setTimeout run, 100

		Template.lfsr.lfsr = ->
			Session.get "lsfrState"

		Template.lfsr.sequence = ->
			Session.get("sequence")?.reverse()

		Template.controls.events =
			"click .step": stepAndUpdateGui
			"click .run": run


		