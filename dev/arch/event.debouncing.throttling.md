# Debouncing And Throttling

`이벤트 그룹화 - 특정 시간 안에서 연속으로 호출된 어떤 이벤트에 대해서 한번만 처리`

## Debouncing

- 이벤트 발생시 실제 처리 시작 시점까지 delay를 두고, delay 시간 내에 중복 이벤트가 발생하면 delay 시간을 초기화 하는 방식
- 일정 시간내에서 마지막 이벤트만 처리
- 반복 생성된 중복 이벤트 중 마지막 이벤트만 처리함

```javascript
const inputElement = document.querySelector('#ID값');
const DELAY = 500;
const handleChange = () => {
  console.log('요청!');
}

let timer;

inputElement.addEventListener('input', () => {
  if (timer) clearTimeout(timer);
  timer = setTimeout(handleChange, DELAY);
});
```

- 예: fuzzy finding 입력 이벤트 중 마지막 입력만 처리

## Throttling

- 이벤트 발생시 이벤트를 처리하고 이후 delay 시간 동안 신규 이벤트를 무시하는 방식
- 일정 시간내에서 첫 이벤트만 처리
- 첫 이벤트를 처리하고 일정시간 동안 발생되는 신규 이벤트를 무시하는 방식

```javascript
const DELAY = 500;
const handleScroll = () => {
  timer = null;
  console.log('스크롤!');
}

let timer;  

window.addEventListener('scroll', () => {
  if (!timer) timer = setTimeout(handleScroll, DELAY);
});
```

