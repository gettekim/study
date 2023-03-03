## 좋아요 동시성 문제에 대한 해결 방안


1. 트랜잭션 격리 수준 설정 예시


  @Service
  public class PostServiceImpl implements PostService {

      @Autowired
      private PostRepository postRepository;

      @Transactional(isolation = Isolation.READ_COMMITTED)
      public void addLike(Long postId) {
          Optional<Post> post = postRepository.findById(postId);
          if (post.isPresent()) {
              Post updatedPost = post.get();
              updatedPost.setLikes(updatedPost.getLikes() + 1);
              postRepository.save(updatedPost);
          } else {
              throw new PostNotFoundException(postId);
          }
      }
  }

    

위 코드에서 @Transactional 어노테이션의 isolation 속성을 READ_COMMITTED로 설정하여 트랜잭션 격리 수준을 설정하고 있습니다.

2. Locking 예시


@Service
public class PostServiceImpl implements PostService {

    @Autowired
    private PostRepository postRepository;

    @Transactional
    public void addLike(Long postId) {
        Optional<Post> post = postRepository.findById(postId, LockModeType.PESSIMISTIC_WRITE);
        if (post.isPresent()) {
            Post updatedPost = post.get();
            updatedPost.setLikes(updatedPost.getLikes() + 1);
            postRepository.save(updatedPost);
        } else {
            throw new PostNotFoundException(postId);
        }
    }
}

위 코드에서 findById 메소드의 두 번째 파라미터로 LockModeType.PESSIMISTIC_WRITE를 사용하여 Pessimistic Locking을 사용하고 있습니다.

3.Optimistic Locking 예시


@Service
public class PostServiceImpl implements PostService {

    @Autowired
    private PostRepository postRepository;

    @Transactional
    public void addLike(Long postId, Long postVersion) {
        Optional<Post> post = postRepository.findById(postId);
        if (post.isPresent()) {
            Post updatedPost = post.get();
            if (updatedPost.getVersion() != postVersion) {
                throw new OptimisticLockException();
            }
            updatedPost.setLikes(updatedPost.getLikes() + 1);
            postRepository.save(updatedPost);
        } else {
            throw new PostNotFoundException(postId);
        }
    }
}

위 코드에서는 Post 엔티티의 버전 관리를 통해 Optimistic Locking을 사용하고 있습니다. 엔티티의 버전이 일치하지 않을 때 OptimisticLockException을 던지고 있습니다.


트랜잭션 격리 수준
장점:

다른 트랜잭션에서 커밋되기 전까지 해당 트랜잭션에서 조회한 데이터를 유지할 수 있습니다.

다른 트랜잭션에서 커밋한 데이터는 반영됩니다.

단점:

트랜잭션을 처리하는 시간이 길어질 수 있습니다.

두 개 이상의 트랜잭션에서 같은 데이터에 동시에 접근하는 경우, 교착 상태(deadlock)가 발생할 수 있습니다.

Locking

2.Pessimistic Locking:

장점:

데이터 충돌이 발생하지 않습니다.
다른 트랜잭션이 해당 데이터에 접근하지 못하게 할 수 있습니다.
단점:

다른 트랜잭션이 기다리는 동안 시간이 지연될 수 있습니다.

너무 많은 트랜잭션이 동시에 Lock을 걸 경우, 데드락(deadlock)이 발생할 수 있습니다.

3.Optimistic Locking:

장점:

데이터 충돌이 발생하지 않습니다.

다른 트랜잭션이 해당 데이터에 접근하지 못하게 할 수 없습니다.

단점:

다른 트랜잭션이 데이터를 업데이트한 경우, 업데이트가 실패할 수 있습니다.

동시에 같은 데이터를 업데이트하는 경우, OptimisticLockException이 발생할 수 있습니다.

트랜잭션 격리 수준과 Locking의 차이점

트랜잭션 격리 수준은 여러 개의 트랜잭션이 동시에 실행될 때, 트랜잭션 간에 데이터를 어떻게 공유할지 결정하는 것입니다.

Locking은 여러 개의 트랜잭션이 동시에 같은 데이터에 접근하는 경우, 충돌을 방지하기 위해 Lock을 걸어서 데이터의 일관성을 유지하는 것입니다.

둘 다 동시성 제어를 위해 사용되지만, 목적이 서로 다릅니다.
