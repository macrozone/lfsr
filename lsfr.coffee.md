

	bits = [true,true,false,true]
	ops = ['not', 'id', 'xor']


	step = ->
		out = bits[0]
		for bit, index in bits[1..]
			bits[index] = doOpt ops[index], bit, out	
		bits[(bits.length-1)] = out

	doOpt = (op, bit, overflowBit) ->
		switch op
			when 'id' then bit
			when 'not' then !bit
			when 'xor' then (!bit) != (!overflowBit)

	sequence = []
	for i in [0..100]
		i = if step() then 1 else 0
		sequence.push i



	