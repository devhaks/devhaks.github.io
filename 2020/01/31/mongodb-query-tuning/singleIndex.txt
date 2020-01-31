
// * 콜렉션 탐색
db.NationalPension.createIndex({ bzowrRgstNo: 1 });
db.NationalPension.find({ bzowrRgstNo: "105879" }).explain('executionStats')

{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "zillinks.NationalPension",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"bzowrRgstNo" : {
				"$eq" : "105879"
			}
		},
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"bzowrRgstNo" : {
					"$eq" : "105879"
				}
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 2007,
		"executionTimeMillis" : 2836,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 3956788,
		"executionStages" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"bzowrRgstNo" : {
					"$eq" : "105879"
				}
			},
			"nReturned" : 2007,
			"executionTimeMillisEstimate" : 2587,
			"works" : 3956790,
			"advanced" : 2007,
			"needTime" : 3954782,
			"needYield" : 0,
			"saveState" : 31004,
			"restoreState" : 31004,
			"isEOF" : 1,
			"invalidates" : 0,
			"direction" : "forward",
			"docsExamined" : 3956788
		}
	},
	"serverInfo" : {
		"host" : "zillinksui-MacBook-Pro.local",
		"port" : 27017,
		"version" : "4.0.5",
		"gitVersion" : "3739429dd92b92d1b0ab120911a23d50bf03c412"
	},
	"ok" : 1
}


// * bzowrRgstNo 인덱스 생성
// https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/
db.NationalPension.createIndex({ bzowrRgstNo: 1 })

{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}

// * bzowrRgstNo 인덱스 삭제
// https://docs.mongodb.com/manual/reference/method/db.collection.dropIndexes/
db.NationalPension.dropIndexes() // Drop all index
db.NationalPension.dropIndexes({ bzowrRgstNo: 1 }) // Drop a specified index

// * 인덱스 목록
db.NationalPension.getIndexes()
	
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "zillinks.NationalPension"
	},
	{
		"v" : 2,
		"key" : {
			"bzowrRgstNo" : 1
		},
		"name" : "bzowrRgstNo_1",
		"ns" : "zillinks.NationalPension"
	}
]

// * 인덱스 적용한 콜렉션 탐색
db.NationalPension.find({ bzowrRgstNo: "105879" }).explain('executionStats')

{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "zillinks.NationalPension",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"bzowrRgstNo" : {
				"$eq" : "105879"
			}
		},
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"bzowrRgstNo" : 1
				},
				"indexName" : "bzowrRgstNo_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"bzowrRgstNo" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"bzowrRgstNo" : [
						"[\"105879\", \"105879\"]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 2007,
		"executionTimeMillis" : 8,
		"totalKeysExamined" : 2007,
		"totalDocsExamined" : 2007,
		"executionStages" : {
			"stage" : "FETCH",
			"nReturned" : 2007,
			"executionTimeMillisEstimate" : 0,
			"works" : 2008,
			"advanced" : 2007,
			"needTime" : 0,
			"needYield" : 0,
			"saveState" : 15,
			"restoreState" : 15,
			"isEOF" : 1,
			"invalidates" : 0,
			"docsExamined" : 2007,
			"alreadyHasObj" : 0,
			"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 2007,
				"executionTimeMillisEstimate" : 0,
				"works" : 2008,
				"advanced" : 2007,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 15,
				"restoreState" : 15,
				"isEOF" : 1,
				"invalidates" : 0,
				"keyPattern" : {
					"bzowrRgstNo" : 1
				},
				"indexName" : "bzowrRgstNo_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"bzowrRgstNo" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"bzowrRgstNo" : [
						"[\"105879\", \"105879\"]"
					]
				},
				"keysExamined" : 2007,
				"seeks" : 1,
				"dupsTested" : 0,
				"dupsDropped" : 0,
				"seenInvalidated" : 0
			}
		}
	},
	"serverInfo" : {
		"host" : "zillinksui-MacBook-Pro.local",
		"port" : 27017,
		"version" : "4.0.5",
		"gitVersion" : "3739429dd92b92d1b0ab120911a23d50bf03c412"
	},
	"ok" : 1
}