---
title: "微服务与单元测试"
date: 2022-10-12T16:05:27+08:00
draft: false
categories: ["ut"]
---

1. 选用[testify](https://github.com/stretchr/testify)，[goconvey](https://github.com/smartystreets/goconvey)来做单测。
2. 选用[mockery](https://github.com/vektra/mockery)来根据go文件里的interface生成service的mockstruct
   1.  mockery --all
3. 选用[go-sqlmock](https://github.com/DATA-DOG/go-sqlmock)来做sqlmock
   1. 使用go-sqlmock+gorm+mysql时，如果使用create，update等非原生方法，需要设置SkipInitializeWithVersion: true，关掉默认事务防止mock失败



直接mockrepo的单测方式，只是测试service层：
```
    func TestConcurrentPost(t *testing.T) {
        Convey("unit testing concurrent_post query service", t, func() {
            logger := log.NewStdLogger(os.Stdout)

            repo := mocks.NewConcurrentPostRepo(t)
            repo.On("QueryConcurrentPost", mock.Anything, mock.AnythingOfType("biz.Option"), mock.AnythingOfType("biz.Option")).Return(&biz.ConcurrentPost{UserId: 1, Id: 1}, nil)

            _hrm := NewHRMServiceService(nil, biz.NewConcurrentPostUsecase(repo, logger), nil, nil, nil, nil, logger)

            _hrm.QueryConcurrentPost(context.Background(), &v1.ReqConcurrentPostId{
                UserId: uint64(1),
                Id:     1,
            })

            repo.AssertExpectations(t)
        })
    }
```

mock sql，可以测试到service层，biz层和data层，但是具体sql并不能测试，
因为sql返回结果是mock的期望值。
```
func TestConcurrentPostWithSql(t *testing.T) {
	Convey("unit testing concurrent_post query service", t, func() {
		logger := log.NewStdLogger(os.Stdout)

		db, sm, err := sqlmock.New(sqlmock.QueryMatcherOption(sqlmock.QueryMatcherEqual))
		if err != nil {
			panic(err)
		}

		sm.ExpectBegin()
		rows := sm.NewRows([]string{"user_id", "id", "department", "post"}).
			AddRow(1, 1, 123, 123)

		sm.ExpectQuery("SELECT * FROM `hrm_concurrent_post` WHERE user_id = ?   and id = ?  ORDER BY `hrm_concurrent_post`.`id` LIMIT 1").
			WillReturnRows(rows)
		sm.ExpectCommit()

		DB, err := gorm.Open(mysql.New(mysql.Config{
			Conn:                      db,
			SkipInitializeWithVersion: true,
		}), &gorm.Config{})
		if err != nil {
			panic(err)
		}
		db.Begin()

		d, cleanup, _ := data.NewData(nil, logger, DB)
		defer cleanup()

		rp := data.NewConcurrentPostRepo(d, logger)
		_hrm := NewHRMServiceService(nil, biz.NewConcurrentPostUsecase(rp, logger), nil, nil, nil, nil, logger)

		ret, err := _hrm.QueryConcurrentPost(context.Background(), &v1.ReqConcurrentPostId{
			UserId: uint64(2),
			Id:     2,
		})

		assert.Nil(t, err)
		fmt.Println(ret.Data)
	})
}

```


思考中：有没有能测试具体sql的单测写法呢？