

**고루틴(goroutine)이란**🤔, 프로그램에 있는 다른 고루틴과 관련하여 독립적으로 동시에 실행되는 함수입니다.

즉, Go 언어로 동시에 실행되는 모든 활동을 고루틴이라고 합니다.

고루틴을 만드는 비용은 스레드에 비해 매우 적기 때문에 경량 스레드라고 합니다.

모든 프로그램은 적어도 하나의 main함수라는 고루틴을 포함하고 고루틴은 항상 백그라운드에서 작동합니다.

메인 함수가 종료되면 모든 고루틴은 종료됩니다. 따라서 고루틴보다 main이 먼저 종료되는 것을 방지해야 합니다



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
