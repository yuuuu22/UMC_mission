// storeController.js

/**
 * 특정 지역에 가게 추가하기 API
 *
 * @swagger
 * /stores:
 *   post:
 *     summary: 특정 지역에 가게 추가
 *     tags:
 *       - Store
 *     requestBody:
 *       description: 가게 이름, 주소, 지역을 포함한 JSON 요청 (Body)
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *                 description: 가게 이름
 *               address:
 *                 type: string
 *                 description: 가게 주소
 *               region:
 *                 type: string
 *                 description: 가게 지역
 *             required:
 *               - name
 *               - address
 *               - region
 *     responses:
 *       201:
 *         description: 가게가 성공적으로 추가됨 (성공 응답)
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 id:
 *                   type: integer
 *                 name:
 *                   type: string
 *                 address:
 *                   type: string
 *                 region:
 *                   type: string
 *       400:
 *         description: 필드 누락 시 실패 (실패 응답)
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 error:
 *                   type: string
 */

const stores = [];

export function createStore(req, res) {
  const { name, address, region } = req.body;

  if (!name || !address || !region) {
    return res.status(400).json({ error: "모든 필드를 입력하세요." });
  }

  const newStore = {
    id: stores.length + 1,
    name,
    address,
    region
  };

  stores.push(newStore);
  res.status(201).json(newStore);
}

/**
 * 가게에 리뷰 추가하기 API
 *
 * @swagger
 * /reviews/{storeId}:
 *   post:
 *     summary: 특정 가게에 리뷰 추가
 *     tags:
 *       - Review
 *     parameters:
 *       - name: storeId
 *         in: path
 *         required: true
 *         description: 리뷰를 작성할 가게 ID (쿼리 파라미터)
 *         schema:
 *           type: integer
 *     requestBody:
 *       description: 리뷰 작성자 ID, 평점, 코멘트 포함 (Body)
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               userId:
 *                 type: string
 *               rating:
 *                 type: number
 *               comment:
 *                 type: string
 *             required:
 *               - userId
 *               - rating
 *     responses:
 *       201:
 *         description: 리뷰 추가 성공 (성공 응답)
 *       404:
 *         description: 가게를 찾을 수 없음 (실패 응답)
 */

const reviews = [];

export function createReview(req, res) {
  const storeId = Number(req.params.storeId);
  const store = stores.find(s => s.id === storeId);
  if (!store) {
    return res.status(404).json({ message: '가게를 찾을 수 없습니다.' });
  }

  const { userId, rating, comment } = req.body;
  const newReview = {
    id: reviews.length + 1,
    storeId,
    userId,
    rating,
    comment
  };
  reviews.push(newReview);
  res.status(201).json(newReview);
}

/**
 * 가게에 미션 추가하기 API
 *
 * @swagger
 * /missions:
 *   post:
 *     summary: 가게에 미션 추가
 *     tags:
 *       - Mission
 *     requestBody:
 *       description: 가게 ID, 미션 제목, 설명 포함 (Body)
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               storeId:
 *                 type: integer
 *               title:
 *                 type: string
 *               description:
 *                 type: string
 *             required:
 *               - storeId
 *               - title
 *               - description
 *     responses:
 *       201:
 *         description: 미션 생성 성공 (성공 응답)
 *       404:
 *         description: 존재하지 않는 가게 (실패 응답)
 */

const missions = [];

export function createMission(req, res) {
  const { storeId, title, description } = req.body;
  const store = stores.find(s => s.id === Number(storeId));
  if (!store) {
    return res.status(404).json({ message: '존재하지 않는 가게입니다.' });
  }

  const newMission = {
    id: missions.length + 1,
    storeId: Number(storeId),
    title,
    description
  };
  missions.push(newMission);
  res.status(201).json(newMission);
}

/**
 * 미션 도전하기 API
 *
 * @swagger
 * /challenges:
 *   post:
 *     summary: 미션 도전하기
 *     tags:
 *       - Challenge
 *     requestBody:
 *       description: 사용자 ID와 도전할 미션 ID 포함 (Body)
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               userId:
 *                 type: string
 *               missionId:
 *                 type: integer
 *             required:
 *               - userId
 *               - missionId
 *     responses:
 *       201:
 *         description: 도전 등록 성공 (성공 응답)
 *       404:
 *         description: 미션을 찾을 수 없음 (실패 응답)
 *       400:
 *         description: 이미 도전 중임 (실패 응답)
 */

const challenges = [];

export function challengeMission(req, res) {
  const { userId, missionId } = req.body;

  const missionExists = missions.some(m => m.id === Number(missionId));
  if (!missionExists) {
    return res.status(404).json({ message: '미션을 찾을 수 없습니다.' });
  }

  const isAlreadyChallenged = challenges.some(c => c.userId === userId && c.missionId === missionId);
  if (isAlreadyChallenged) {
    return res.status(400).json({ message: '이미 도전 중입니다.' });
  }

  const newChallenge = {
    id: challenges.length + 1,
    userId,
    missionId: Number(missionId)
  };
  challenges.push(newChallenge);
  res.status(201).json(newChallenge);
}
