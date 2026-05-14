---
title: ใช้ functional options pattern เข้ามาช่วยในการเขียน unit test
draft: false
description: ""
slug: ใช้-functional-options-pattern-เข้ามาช่วยในการเขียน-unit-test
date: 2026-05-06 00:00:00+0000
started_at: 2026-05-05 00:00:00+0000
image: IMG_20260422_064837.jpg
categories:
    - devlog
tags: ["golang"]
weight: 1
---
การเขียน unit test เป็นสิ่งหนึ่งที่ต้อง initialize service ต่างๆตามเคสที่เราต้องการเทสอยู่เสมอ
โดยปกติการ init service ส่วนใหญ่จะ init เฉพาะสิ่งที่เราต้องการใช้ในเคสนั้นๆ เท่านั้น
และส่ง ```nil``` หรือ ค่า default เป็น parameter เข้าไปแทนในตำแหน่งของ service ที่ไม่เกี่ยวข้องกับเทสเคส
```go
// user_service.go
type identityService struct {
        userRepo repo.User
        timeLeaveRepo repo.TimeLeave
        notificationRepo repo.Notification
}

// user_service_test.go
func newUserService(userRepo repo.User, timeLeaveRepo repo.TimeLeave, notificationRepo repo.Notification) *identityService {
        return &identityService{
                userRepo: userRepo,
                timeLeaveRepo, timeLeaveRepo,
                notificationRepo: notificationRepo,
        }
}


func Test_Fail_CreateNewUser(t *testing.T) {
        userRepo := repo.MockUserRepo{}
        svc := newUserService(userRepo, nil, nil) // <- ต้องส่งทุก repository ให้ครบ
        _, err := svc.CreateNewUser(payload)

        assert.Nil(t, err)
}
```

ลองนำ functional options pattern มาประยุกค์ใช้ เพื่อลดการส่ง ```nil``` เข้าไปในตำแหน่งที่เราไม่ต้องการใช้
```go
type userServiceOption func(*identityService)

func withUserRepo(repo repo.User) userServiceOption {
        return func(s *identityService) {
                s.userRepo = repo
        }
}

func withTimeLeaveRepo(repo repo.TimeLeave) userServiceOption {
        return func(s *identityService) {
                s.timeLeaveRepo = repo
        }
}

func withNotificationRepo(repo repo.Notification) userServiceOption {
        return func(s *identityService) {
                s.notificationRepo = repo
        }
}

func newUserService(opts ...userServiceOption) *identityService {
	userSvc := identityService{}

	for _, opt := range opts {
		opt(&userSvc)
	}

	return &userSvc
}

func Test_Fail_CreateNewUser(t *testing.T) {
        userRepo := repo.MockUserRepo{}
        svc := newUserService(withUserRepo(userRepo)) // <- เลือกส่งเฉพาะ repository ที่ต้องการ
        _, err := svc.CreateNewUser(payload)

        assert.Nil(t, err)
}
```




