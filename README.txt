�}�X�^�e�[�u���̃f�[�^�͕p�ɂɕς��Ȃ����߁A�L���b�V���Ƃ��ă�������ɕێ����悤�Ǝv���܂��B
���܂ł͂���肷�郊�|�W�g������entities��ێ����Ă��܂����B

internal sealed class TestRepository : ITestEntityRepository
{
    private readonly IReadOnlyList<TestEntity> _entities = [];

    public IReadOnlyList<TestEntity> GetAll()
    {
        throw new NotImplementedException();
    }

    public TestEntity GetById(int id)
    {
        throw new NotImplementedException();
    }
}

�������A�h���C���쓮�J���̍u�`�ł͈ȉ��̂悤�ɂ��������悢�Ƃ���Ă��܂����B

public static class Tests
{
    private static List<TestEntity> _entities = [];

    public static void Create(ITestEntityRepository repository)
    {
        lock(((ICollection)_entities).SyncRoot)
        {
            _entities.Clear();

            _entities.AddRange(repository.GetAll());
        }
    }

    public static TestEntity? GetById(int id)
    {
        lock (((ICollection)_entities).SyncRoot)
        {
            return _entities.FirstOrDefault(entity => entity.UserId == id);
        }
    }
}

��������Ă������������ꍇ�ɁAViewModel�ł͂ǂ̂悤�ɂ���΂悢�ł��傤���H



-----



public class TestViewModel : FormViewModelBase
{
    private readonly ITestEntityRepository _repository;

    public TestViewModel(IMessageBoxService messageBoxService, ITestEntityRepository repository)
        : base(messageBoxService)
    {
        _repository = repository;

        // �L���b�V����������
        InitializeCache();
    }

    private void InitializeCache()
    {
        // ���|�W�g������f�[�^���擾���A�L���b�V�����쐬
        Tests.Create(_repository);
    }

    // UI�ŕ\������}�X�^�f�[�^���X�g
    public IReadOnlyList<TestEntity> MasterData => Tests.GetAll();

    // ID����G���e�B�e�B���擾
    public TestEntity? GetEntityById(int id)
    {
        return Tests.GetById(id);
    }

    // ���t���b�V�������i�Ⴆ�΁A�蓮�X�V�p�j
    public void RefreshCache()
    {
        try
        {
            // �L���b�V���̍X�V
            Tests.Create(_repository);
            this.StatusLableText = "�L���b�V�����X�V���܂���";
        }
        catch (Exception ex)
        {
            _messageBoxService.ShowErrorMessage($"�L���b�V���X�V���ɃG���[���������܂���: {ex.Message}");
        }
    }
}



-----