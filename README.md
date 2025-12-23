P2PK 주소나 P2PK 보상방식을 처리하기위해 파이썬 API로 중간 브릿지 형성 <br>
pip install fastapi uvicorn requests  <br>
uvicorn proxy:app --host 0.0.0.0 --port 8888  <br>
혹은 PM2로 구동 <br>
이전 버전에 비해 esplora-nginx.conf 수정됨 <br>
generate py를 통해 스캐닝을 먼저 해줘야함 <br>


<br><br><br><br><br><br><br><br><br><br><br><br>

###Web Title , Ticker Change###

cd ~/bitever-esplora
git clone https://github.com/Blockstream/esplora.git frontend
cd frontend

# 환경변수로 API 엔드포인트 지정 (/api로 리버스프록시할 계획이라면 굳이 지정 안 해도 기본이 /api)
# 예: electrs 컨테이너의 HTTP(REST) 포트가 3000이라면 Nginx에서 /api -> electrs:3000 프록시를 둘 예정
export API_URL=/api

npm install
npm run dist
ls -l dist/index.html   # <- 요게 생겨야 정상입니다

도커 엔진엑스 설정 리로드
docker exec -it bitever-esplora-nginx nginx -t
docker exec -it bitever-esplora-nginx nginx -s reload


electrs REST API 오픈 확인
curl -i http://127.0.0.1:3002/api/blocks/tip/height
curl -i http://127.0.0.1:3002/api/mempool/recent


electrs 페이지 인덱싱 진행률 보기   (상당히 걸림: 초기인덱싱tx, 압축인덱싱history, ) 용량으로 치면 노드 데이터의 각각 1배수씩 먹는다고보면됨
1) electrs 인덱싱 진행률 모니터
# 진행 로그만 추려 보기 (어느 정도 올라오는지)
docker logs -f electrs | grep -E 'Tx indexing is up to height|BlockchainInfo|REST server|listening on'



# 에스플로라 티커 단위 바꾸기
cd ~/bitever-esplora/frontend/dist
 #우선어디에들어있는지확인
grep -R --line-number -I '"BTC"' .
grep -R --line-number -I '>BTC<' .
# 백업후치환(대문자 "BTC"만대상)
find . -type f \( -name '*.js' -o -name '*.html' \) -print0 \
 | xargs -0 sed -i.bak 's/"BTC"/"BEC"/g'
 # HTML 태그사이텍스트로있는경우도치환
find . -type f -name '*.html' -print0 \
 | xargs -0 sed -i.bak 's/>BTC</>BEC</g'
