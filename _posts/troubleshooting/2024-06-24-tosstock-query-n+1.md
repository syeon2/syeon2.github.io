---
layout: post
title: "[Troubleshooting] JPA의 N + 1 Query issue 성능 개선 사례"
subtitle: "JPA에서 발생하는 N + 1 이슈를 QueryDSL과 DTO를 사용하여 해결한 사례를 소개합니다."
categories: devlog
tags: troubleshooting jpa n+1 querydsl
---

> JPA에서 발생하는 N + 1 이슈를 QueryDSL과 DTO를 사용하여 성능을 약 85% 개선시킨 사례를 소개합니다..

<!--more-->

- 목차
  - [문제 인식](#-문제-인식)
  - [트러블 슈팅](#-트러블슈팅)
  - [개선-방안](#-개선-방안)
  - [결론](#-결론)

## 🌱 문제 인식

tosstock 프로젝트에서는 증권 별로 게시글을 작성할 수 있는 기능을 가지고 있습니다.

각 게시글 데이터로는 회원 아이디, 회원 이름, 게시글 내용, 게시글 아이디, 현 게시글을 좋아요 누른 개수, 현 게시글 총 댓글 수, 게시글 생성일, 게시글 수정일이 있습니다.

게시글 엔티티는 회원 엔티티와 연관관계를 맺고 있어 별다른 쿼리 요청을 하지 않고, 현 게시글을 좋아요 누른 개수와 총 댓글 수를 구하기 위한 별도의 쿼리 처리를 진행하도록 했습니다. 이 과정에서 성능 문제를 발견했습니다.

<a href="https://ibb.co/S5k04b1"><img src="https://i.ibb.co/17CKt1p/prev-n-1issue2.png" alt="prev-n-1issue2" border="0"></a>

게시글 리스트를 조회하는테 총 5개의 쿼리가 발생한 것을 볼 수 있습니다. 각 쿼리는 다음과 같습니다.

1. 게시글 전체 조회 쿼리
2. Pageable 인터페이스 사용시 함께 요청되는 count 쿼리
3. 현 게시글을 좋아요 누른 데이터 count 쿼리
4. 현 게시글에 대한 댓글 count 쿼리
5. 회원 조회 쿼리???

2, 3, 4번과 다르게 5번(회원 조회) 쿼리는 의도하지 않은 쿼리였습니다.

게시글 엔티티에 있는 회원 필드로부터 게시글 DTO에 필요한 회원 아이디와 이름을 넣기 때문에 의도적으로 회원 정보를 조회하지 않았지만, 회원 정보를 요청하는 쿼리가 생성되었습니다.

JPA에서는 흔히 N + 1이라고 하는 쿼리 이슈였고, 해당 이슈를 해결함으로써 성능 최적화를 진행해보려합니다.


## 🌱 원인 분석

JPA의 N + 1 문제는 해당 연관관계를 맺은 방법에서 발생합니다.

```java
@Getter
@Entity
@Table(name = "post")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class PostEntity extends BaseEntity {
	
    //...
    @Setter
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private MemberEntity member;
    //...
}
```

게시글 엔티티와 회원 엔티티는 ManyToOne으로 연관관계가 맺어져있습니다. 이 때, 주의해야할 부분은 `fetch = FetchType.LAZY` 입니다.

LAZY 옵션을 사용하면 현재 엔티티를 조회할 때 연관된 엔티티는 바로 로딩되지 않고, 실제로 해당 연관 엔티티의 속성에 접근할 때 쿼리가 실행되어 로딩됩니다. 이는 필요한 경우에만 데이터를 로딩하여 초기 로딩 속도를 개선하지만, 잘못 사용하면 N + 1 문제를 초래할 수 있습니다.

<a href="https://ibb.co/VVmJmhx"><img src="https://i.ibb.co/R2yYyMN/40-A16-D28-EE42-4163-9384-C450-E09-E6-DAA.jpg" alt="40-A16-D28-EE42-4163-9384-C450-E09-E6-DAA" border="0"></a>

위의 그림과 같이 Post 엔티티를 조회하면 Post 엔티티 데이터만 조회하고 Post Entity의 member 필드의 값 중 하나라도 (id 제외) 사용하게 되면 그 즉시 해당 id를 가진 member 엔티티를 조회하게 됩니다. 
(조회할 때는 JPA의 특징인 Entity Manager를 거치게 되고, 이미 Entity Manager의 1차 캐시에 해당 엔티티가 캐싱되어 있다면 DB에 쿼리 요청을 하지 않고 캐싱된 데이터를 가지고 옵니다.)

<strong> 그럼 LAZY를 사용하지 않고 EAGER를 사용해서 Post 엔티티를 조회할 때마다 member를 join해서 가지고 오면 되지 않을까?</strong> 이 상황은 더 큰 문제를 야기합니다.

EAGER 옵션을 사용하면 모든 연관 엔티티를 즉시 로딩하게 되는데, 이는 연관된 엔티티가 많을 경우 복잡한 쿼리를 발생시켜 성능 저하를 초래할 수 있습니다. 예를 들어, Post 엔티티를 조회할 때 Member, Comment, Like 등 여러 엔티티가 동시에 로딩된다면, 매우 복잡한 SQL 조인이 발생하여 성능이 저하됩니다.

LAZY의 N + 1 이슈를 방지하기 위해 LAZY 옵션을 모두 EAGER로 변경한다면, Post 엔티티를 조회할 때 Member 엔티티도 함께 조회합니다. 하지만, Member 엔티티에 또 다른 엔티티가 EAGER로 연관관계 맺고 있다면... 그야말로 총체적 난국 쿼리가 만들어질 것입니다.

<a href="https://ibb.co/Kzy9VBv"><img src="https://i.ibb.co/DgK7wm2/ex-Post-Member-Comment.jpg" alt="ex-Post-Member-Comment" border="0"></a>
(😵 연관관계 맺고 있는 모든 엔티티들을 Join..)

-----

## 🌱 트러블슈팅

> 💡 N + 1 이슈를 해결하기 위해서는 JPQL에서 지원하는 fetch join을 사용하는 것이 일반적입니다.
> 
> join 절에 fetch 키워드를 추가하여 조회하려는 엔티티와 연관관계 맺어져있는 엔티티를 함께 조회하고 영속성 컨텍스트에 영속화합니다. (QueryDSL에도 지원)

기존 이슈는 N + 1 외에도 여러 가지 최적화할 사항이 존재합니다.

1. count 쿼리가 별도로 요청되어지고 있는 상황
2. page를 사용함으로 count 쿼리가 자동으로 발생하고 있는 상황
3. N + 1 이슈

DTO를 새롭게 만들어 조회하는 방법은 위의 이슈를 한번에 해결할 수 있는 방법이라 판단하여 프로젝트에 적용하였습니다. (QueryDSL를 사용합니다.)

### 🥕 기존 코드
```java
//...
@Getter
@Entity
@Table(name = "post")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class PostEntity extends BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id", columnDefinition = "bigint", nullable = false)
    private Long id;
    
    @Lob
    @Column(name = "article", columnDefinition = "text", nullable = false)
    private String article;
    
    @Setter
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private MemberEntity member;
    
    @Setter
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "stock_id")
    private StockEntity stock;
    
    @OneToMany(
        mappedBy = "post",
        cascade = CascadeType.REMOVE,
        orphanRemoval = true,
        fetch = FetchType.LAZY)
    private List<CommentEntity> comments = new ArrayList<>();
    
    @Builder
    private PostEntity(Long id, String article, MemberEntity member, StockEntity stock) {
        this.id = id;
        this.article = article;
        this.member = member;
        this.stock = stock;
    }
}

//----------------------------------

@Override
@Transactional(readOnly = true)
public List<MainBoardPostDto> findPostByStockId(Long stockId, Pageable pageable) {
    Page<PostEntity> findPosts = postRepository.findByStockId(stockId, pageable);
    
    return findPosts.getContent().stream()
        .map(post -> postMapper.toMainPostDto(
            post, 
            postLikeRepository.countByPostId(post.getId()),
            commentRepository.countByPostId(post.getId())))
        .collect(Collectors.toList());
}

//...
```

기존 코드는 사실상 간단했습니다. stockId를 통해 JpaRepository 인터페이스를 구현한 Impl Repository로부터 findByStockId 메서드를 만들어 사용했습니다.

findByStockId 메서드로 조회한 게시글 데이터 각각 postLike count와 comment count를 요청하는 쿼리를 요청합니다.

위의 메서드를 통해 요청되는 쿼리는 다음과 같습니다.

1. 특정 Stock Id를 갖는 게시글 리스트를 조회하는 쿼리
2. Pageable로 인해 자동으로 count 조회하는 쿼리
3. 각각 게시글마다 Like 개수 count 조회하는 쿼리
4. 각각 게시글마다 comment 개수 count 조회하는 쿼리
5. 각각 게시글 작성자 Member 조회 쿼리 (N + 1)

약 20개의 게시글 데이터를 조회할 때의 성능입니다.

#### 📉 트러블슈팅 전 성능

<a href="https://ibb.co/S5k04b1"><img src="https://i.ibb.co/17CKt1p/prev-n-1issue2.png" alt="prev-n-1issue2" border="0"></a>

<a href="https://ibb.co/8cTWTmx"><img src="https://i.ibb.co/ySGwGNk/prev-n-1issue1.png" alt="prev-n-1issue1" border="0"></a>

쿼리는 보이기에 5개의 쿼리만 동작하는 것으로 보이지만, 각 게시글 데이터마다 좋아요 count 쿼리와 댓글 수 count 쿼리가 추가적으로 발생합니다. 또한, 각자 다른 회원들이 게시글을 작성했다면 각각 회원을 조회하는 
쿼리가 발생하기 때문에 20개의 게시글을 가지고 오는 API에 최대 62개의 쿼리 요청이 발생될 수 있습니다.

서버는 DB와 빠르고 안전하게 (OOME 방지) 통신하기 위해 지정된 개수의 커넥션을 만들어 사용합니다.(커넥션 풀) 이 상황에서 한 API당 62개의 쿼리를 요청하게 되면 쉽게 커넥션 풀의 커넥션이 고갈되어 병목현상을 유발할 수 있습니다.

이를 방지하기 위해 트러블슈팅이 꼭 필요한 상황이라 판단하고 빠르게 개선하고자 하였습니다.

---

### 🥕 개선된 코드

```java
//......
@Override
public List<MainBoardPostDto> findMainBoardPostDtoByStockId(Long stockId, Long offset, Long limit, String sort) {
    return queryFactory
        .select(Projections.constructor(MainBoardPostDto.class,
            QPostEntity.postEntity.id,
            Projections.constructor(MemberDto.class,
            QMemberEntity.memberEntity.id,
            QMemberEntity.memberEntity.username
        ),
        QPostEntity.postEntity.article,
        QPostLikeEntity.postLikeEntity.count().intValue(),
        QCommentEntity.commentEntity.count().intValue(),
        QPostEntity.postEntity.createdAt,
        QPostEntity.postEntity.updatedAt))
        .from(QPostEntity.postEntity)
        .join(QPostEntity.postEntity.member, QMemberEntity.memberEntity)
        .leftJoin(QPostLikeEntity.postLikeEntity)
        .on(QPostLikeEntity.postLikeEntity.post.id.eq(QPostEntity.postEntity.id))
        .leftJoin(QCommentEntity.commentEntity)
        .on(QCommentEntity.commentEntity.post.id.eq(QPostEntity.postEntity.id))
        .where(QPostEntity.postEntity.stock.id.eq(stockId))
        .where((sort.equalsIgnoreCase("desc") ? QPostEntity.postEntity.id.lt(offset) :
        QPostEntity.postEntity.id.gt(offset)))
        .groupBy(QPostEntity.postEntity.id, QMemberEntity.memberEntity.id, QMemberEntity.memberEntity.username,
            QPostEntity.postEntity.article, QPostEntity.postEntity.createdAt, QPostEntity.postEntity.updatedAt)
        .orderBy((sort.equalsIgnoreCase("desc") ? QPostEntity.postEntity.createdAt.desc() :
            QPostEntity.postEntity.createdAt.asc()))
        .limit(limit)
        .fetch();
}

```

Member Entity로 인한 fetch join, post Like와 Comment 테이블을 join한 후 count 값을 반환해야 하는 여러 이슈들을 해결하기 위해 DTO를 사용하여 값을 조회하도록 수정했습니다.

QueryDSL을 사용하면 타입 안전한 쿼리를 작성할 수 있으며, 코드의 가독성이 높아지고 컴파일 타임에 쿼리 오류를 잡아낼 수 있어 유지 보수성이 향상됩니다. 아래 QueryDSL 코드는 다음 SQL문을 생성합니다.

```mysql
SELECT
    p.post_id,
    p.article,
    p.created_at,
    p.updated_at,
    m.member_id,
    m.username,
    count(pl.post_like_id) as count_post_like,
    count(c.comment_id) as count_post_comment
FROM
    post p
JOIN
    member m
    ON m.member_id = p.member_id
LEFT JOIN
    post_like pl
    ON pl.post_id = p.post_id
LEFT JOIN
    comment c
    on c.post_id = p.post_id
WHERE
    p.stock_id = ${stockId}
    AND
    p.post_id < ${offset}
GROUP BY
    p.post_id, p.article, p.created_at, p.updated_at, m.member_id, m.username
ORDER BY
    p.post_id DESC
LIMIT 20

```

#### 📈 트러블슈팅 후 성능

<a href="https://ibb.co/xC7tkXg"><img src="https://i.ibb.co/5MRg0Gj/post-n-1issue2.png" alt="post-n-1issue2" border="0"></a>

<a href="https://ibb.co/jrsrjfd"><img src="https://i.ibb.co/9rSrdgJ/post-n-1issue1.png" alt="post-n-1issue1" border="0"></a>

해당 로직을 통해 위의 최대 62개 생성되는 쿼리를 단 1개로 줄일 수 있었고 쿼리 속도 또한 빠르게 개선할 수 있었습니다. (평균 600ms -> 80 ~ 90ms -> 기존으로부터 약 80%의 속도를 절감)

이로 인해 전체 쿼리 실행 시간이 크게 단축되었으며, 데이터베이스 부하도 줄어들어 더 많은 요청을 효율적으로 처리할 수 있게 되었습니다. 이를 통해 시스템의 전반적인 성능이 향상되었습니다.

---

## 🌱 결론

이번 트러블슈팅을 통해 쿼리 최적화의 중요성을 깨달았습니다. 같은 기능이라도 성능이 크게 달라질 수 있으며, 이는 해당 기술에 대한 깊은 이해가 바탕이 되어야 함을 느꼈습니다.

아키텍처 레벨에서 프로덕트를 최적화하기 이전에 코드 레벨에서 빠르고 효과적인 최적화 방안들을 고민하고 적용해보아야겠다는 생각이 드는 트러블슈팅이었습니다.