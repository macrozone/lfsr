	
	if Meteor.isClient

		sampleConfig = 
			bits: [true,false,true,false,false,false,false]
			ops: ['not', 'id', 'xor','not','id','not']


		sampleConfig2 = 
			bits: [false,false,true]
			ops: ['not', 'id', 'xor','not','id','not']

			#bits: [true,false,false,false,false,false,false]
			#ops: ['not', 'id', 'xor','xor','id','not']
			#ops: ['id', 'id', 'id','id','id', 'id','id','id', 'id','id','id', 'id','id','id', 'id','id']
		
		

		
		


		setupExperiment sampleConfig
		

			