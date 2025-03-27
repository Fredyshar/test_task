# test_task
Тестовое задание


Задание 4.

```
python
import requests
import pytest
import re

BASE_URL = "https://jsonplaceholder.typicode.com"
POST_ID = 1
EMAIL_PATTERN = r"[^@\s]+@[^@\s]+\.[a-z]+"
VALID_COMMENT = {"name": "Toster", "email": "toster@moster.com", "body": "This is a test comment"}
INVALID_COMMENTS = [
    ({"name": "", "email": "user@example.com", "body": "Text"}, "name"),
    ({"email": "user@example.com", "body": "Text"}, "name"),
    ({"name": "User", "email": "notanemail", "body": "Text"}, "email"),
    ({"name": "User", "body": "Text"}, "email"),
    ({"name": "User", "email": "user@example.com"}, "body"),
    ({"name": "User", "email": "user@example.com", "body": ""}, "body"),
]


def get_comments_endpoint(post_id):
    return f"{BASE_URL}/posts/{post_id}/comments"


def test_get_all_post_status_code():
    response = requests.get(f"{BASE_URL}/posts")
    assert response.status_code == 200
    response_data = response.json()
    assert isinstance(response_data, list)


def test_get_comments_status_code():
    response = requests.get(get_comments_endpoint(POST_ID))
    assert response.status_code == 200


def test_get_comments_structure():
    response = requests.get(get_comments_endpoint(POST_ID))
    assert response.status_code == 200
    comments = response.json()
    assert isinstance(comments, list)
    for comment in comments:
        assert "id" in comment
        assert "postId" in comment and comment["postId"] == POST_ID
        assert "name" in comment and comment["name"]
        assert "email" in comment and re.match(EMAIL_PATTERN, comment["email"])
        assert "body" in comment and comment["body"]


def test_post_comment_success():
    response = requests.post(get_comments_endpoint(POST_ID), json=VALID_COMMENT)
    assert response.status_code == 201
    response_data = response.json()
    for key in VALID_COMMENT:
        assert response_data[key] == VALID_COMMENT[key]
    assert "id" in response_data, "ID должен возвращаться при создании комментария"


@pytest.mark.xfail(reason="jsonplaceholder не валидирует обязательность поля")
@pytest.mark.parametrize("request_data, missing_field", INVALID_COMMENTS)
def test_post_comment_required_fields(request_data, missing_field):
    response = requests.post(get_comments_endpoint(POST_ID), json=request_data)
    assert response.status_code == 400  # jsonplaceholder возвращает 201


@pytest.mark.xfail(reason="jsonplaceholder не валидирует формат email")
def test_post_comment_with_invalid_email_format():
    request_data = {"name": "toster", "email": "@toster", "body": "This is a test comment"}
    response = requests.post(get_comments_endpoint(POST_ID), json=request_data)
    assert response.status_code == 400
    email = request_data["email"]
    assert re.fullmatch(EMAIL_PATTERN, email), f"Email некорректный: {email}"
```
