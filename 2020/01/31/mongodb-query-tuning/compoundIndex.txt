
db.NationalPension.createIndex({ bzowrRgstNo: 1, adptDt: 1 });
db.NationalPension.createIndex({ bzowrRgstNo: 1, adptDt: 1, wkplNm: "text" });


db.NationalPension.find({
	adptDt: new ISODate("1988-01-01T00:00:00.000Z"),
	bzowrRgstNo: "101810",
}).explain('executionStats').executionStats

	//
	db.NationalPension.find({
		wkplNm: /어반/,
		wkplJnngStcd: 1
	}).sort({
			wkplNm: 1
		}).explain('executionStats').executionStats
	
	// 본문 검색
db.NationalPension.find({
	bzowrRgstNo: "101810",
	adptDt: {
		$gt: new ISODate("1988-01-01T00:00:00.000Z")
	},
	$text: { $search: '어반' }
}).explain('executionStats').executionStats


db.NationalPension.createIndex({ dataCrtYm: 1, bzowrRgstNo: 1 }); 
db.NationalPension.createIndex({ bzowrRgstNo: 1, dataCrtYm: 1 });

db.NationalPension.find({ $text: { $search: "어반플레이" } })

db.NationalPension.find({
	dataCrtYm: new ISODate("2019-11-01T00:00:00.000Z"),
}).sort({
		bzowrRgstNo: -1
	}).explain('executionStats').executionStats.executionTimeMillis

// > "executionTimeMillis" : 1294,

db.NationalPension.find({
	bzowrRgstNo: "105879",
}).sort({
	dataCrtYm: 1
}).explain('executionStats').executionStats.executionTimeMillis

// > ".executionStats.executionTimeMillis" : 12714


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
	},
	{
		"v" : 2,
		"key" : {
			"bzowrRgstNo" : 1,
			"adptDt" : 1
		},
		"name" : "bzowrRgstNo_1_adptDt_1",
		"ns" : "zillinks.NationalPension"
	}
]

// * 콜렉션 탐색
db.NationalPension.find({
  wkplNm:"주식회사어반플레이", 
  bzowrRgstNo: "105879", 
  adptDt: {
    $gte: new ISODate("2013-12-20T00:00:00.000Z")
  }

// db.NationalPension.find({
//   bzowrRgstNo: "105879", 
//   adptDt: {
//     $gte: new ISODate("2013-12-20T00:00:00.000Z")
//   }
// }).explain('executionStats')

{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "zillinks.NationalPension",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [
				{
					"bzowrRgstNo" : {
						"$eq" : "105879"
					}
				},
				{
					"wkplNm" : {
						"$eq" : "주식회사어반플레이"
					}
				},
				{
					"adptDt" : {
						"$gte" : ISODate("2013-12-20T00:00:00Z")
					}
				}
			]
		},
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"$and" : [
					{
						"bzowrRgstNo" : {
							"$eq" : "105879"
						}
					},
					{
						"wkplNm" : {
							"$eq" : "주식회사어반플레이"
						}
					},
					{
						"adptDt" : {
							"$gte" : ISODate("2013-12-20T00:00:00Z")
						}
					}
				]
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 8,
		"executionTimeMillis" : 2941,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 3956788,
		"executionStages" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"$and" : [
					{
						"bzowrRgstNo" : {
							"$eq" : "105879"
						}
					},
					{
						"wkplNm" : {
							"$eq" : "주식회사어반플레이"
						}
					},
					{
						"adptDt" : {
							"$gte" : ISODate("2013-12-20T00:00:00Z")
						}
					}
				]
			},
			"nReturned" : 8,
			"executionTimeMillisEstimate" : 2666,
			"works" : 3956790,
			"advanced" : 8,
			"needTime" : 3956781,
			"needYield" : 0,
			"saveState" : 31010,
			"restoreState" : 31010,
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
db.NationalPension.createIndex({ bzowrRgstNo: 1, adptDt: 1   })
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 2,
	"numIndexesAfter" : 3,
	"ok" : 1
}
// wkplNm:"주식회사어반플레이", 
//   bzowrRgstNo: "105879", 
//   adptDt: {
//     $gte: new ISODate("2013-12-20T00:00:00.000Z")
// }

// * bzowrRgstNo 인덱스 삭제
// https://docs.mongodb.com/manual/reference/method/db.collection.dropIndexes/
db.NationalPension.dropIndexes() // Drop all index
db.NationalPension.dropIndex({ bzowrRgstNo: 1 }) // Drop a specified index
db.NationalPension.dropIndex({ bzowrRgstNo: 1, adptDt: 1 }) // Drop a specified index
