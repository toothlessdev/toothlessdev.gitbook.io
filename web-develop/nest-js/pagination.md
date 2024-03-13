# Pagination

Pagination : 많은 데이터를 부분적으로 나눠서 불러오는 기술

쿼리에 해당하는 모든 데이터를 한번에 다 불러오지 않고 부분적으로 쪼개서 불러온다



Page Based Pagination

페이지를 기준으로 데이터를 잘라서 요청하는 Pagination

요청을 보낼때 원하는 데이터 개수와 몇번째 페이지를 가져올지 명시

페이지 숫자를 누르면 다음 페이지로 넘어가는 형태의 UI 에서 많이 사용

Pagination 도중에 데이터베이스에서 데이터가 추가되거나 삭제될 경우 저장되는 데이터가 누락되거나 중복될 수 있음

Pagination 알고리즘이 매우 간단함



Cursor Based Pagination

