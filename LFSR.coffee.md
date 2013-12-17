	
	@LFSR = class 
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