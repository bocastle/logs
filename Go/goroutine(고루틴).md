

**고루틴(goroutine)이란**🤔, 프로그램에 있는 다른 고루틴과 관련하여 독립적으로 동시에 실행되는 함수입니다.

즉, Go 언어로 동시에 실행되는 모든 활동을 고루틴이라고 합니다.

고루틴을 만드는 비용은 스레드에 비해 매우 적기 때문에 경량 스레드라고 합니다.

모든 프로그램은 적어도 하나의 main함수라는 고루틴을 포함하고 고루틴은 항상 백그라운드에서 작동합니다.

메인 함수가 종료되면 모든 고루틴은 종료됩니다. 따라서 고루틴보다 main이 먼저 종료되는 것을 방지해야 합니다

#### 모델 정의 (`model/sample_model.go`)

```go
package model

type ReqSampleDetail struct {
	SampleNo int `form:"sample_no"`
}

type SampleDetail struct {
	No           int    `json:"no"`
	DummyTitle   string `json:"dummy_title"`
	DummyContent string `json:"dummy_content"`
	ViewCount    int    `json:"view_count"`
}

```
#### SQL 처리 (`sqlmap/sample_sqlmap.go`)

```go

package sqlmap

import (
	"database/sql"
	"yourproject/model"
)

func IncreaseSampleViewCount(db *sql.DB, sampleNo int) error {
	query := `
		UPDATE sample_table
		SET view_count = view_count + 1
		WHERE no = ?
	`
	_, err := db.Exec(query, sampleNo)
	return err
}

func GetSampleDetail(db *sql.DB, sampleNo int) (model.SampleDetail, error) {
	query := `
		SELECT no, dummy_title, dummy_content, view_count
		FROM sample_table
		WHERE no = ?
	`
	var res model.SampleDetail
	err := db.QueryRow(query, sampleNo).Scan(&res.No, &res.DummyTitle,    &res.DummyContent, &res.ViewCount)
	return res, err
}

```
#### 서비스 로직 (`service/sample_service.go`)

```go

package service

import (
	"sync"
	"time"

	"yourproject/model"
	"yourproject/sqlmap"
	"database/sql"
)

type SampleService struct {
	db       *sql.DB
	history  map[string]time.Time
	mu       sync.Mutex
}

func NewSampleService(db *sql.DB) *SampleService {
	return &SampleService{
		db:      db,
		history: make(map[string]time.Time),
	}
}

func (s *SampleService) shouldIncreaseView(key string) bool {
	s.mu.Lock()
	defer s.mu.Unlock()

	// 1분 이내 중복조회 방지
	if t, ok := s.history[key]; ok {
		if time.Since(t) < time.Minute {
			return false
		}
	}

	s.history[key] = time.Now()
	return true
}

func (s *SampleService) GetSampleDetail(sampleNo int, clientIP string) (model.SampleDetail, error) {
	key := clientIP + "-" + string(rune(sampleNo))

	if s.shouldIncreaseView(key) {
		go func() {
			_ = sqlmap.IncreaseSampleViewCount(s.db, sampleNo)
		}()
	}

	return sqlmap.GetSampleDetail(s.db, sampleNo)
}
```
#### 사용 예시 (핸들러에서)

```go

func GetSampleDetail(c *gin.Context) {
	var req model.ReqSampleDetail
	if err := c.ShouldBindQuery(&req); err != nil {
		handleError(c, 400, "잘못된 파라미터", nil)
		return
	}

	clientIP := c.ClientIP()
	res, err := sampleService.GetSampleDetail(req.SampleNo, clientIP)
	if err != nil {
		handleError(c, 500, "조회 실패", nil)
		return
	}

	ResponseMSG(c, 200, "성공", res)
}
```
