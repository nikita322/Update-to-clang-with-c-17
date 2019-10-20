Update to clang with c++17
=============================
1. Replace "g++" or "g++48"/"g++49" to "c++"

2. Raplace "-mtune=i686" or "-mcpu=i686" to "-m32"

3. Add new flag for all Makefile "-std=c++17"

4. Go to game/cipher.h replace this:  
    `encoder_->ProcessData((byte*)buffer, (const byte*)buffer, length);`  
on this:  
    `encoder_->ProcessData((CryptoPP::byte*)buffer, (const CryptoPP::byte*)buffer, length);`  
and:  
    `decoder_->ProcessData((byte*)buffer, (const byte*)buffer, length);`  
on this:  
    `decoder_->ProcessData((CryptoPP::byte*)buffer, (const CryptoPP::byte*)buffer, length);`

5. Go to common/stl.h remove that:  
```cpp
template <typename T> T MIN(T a, T b)
{
    return a < b ? a : b;
}

template <typename T> T MAX(T a, T b)
{
    return a > b ? a : b;
}
template <typename T> T MINMAX(T min, T value, T max)
{
    T tv;

    tv = (min > value ? min : value);
    return (max < tv) ? max : tv;
}
```  
    And this:  
        ```
        #ifdef __GNUC__
        #include <ext/functional>
        #endif
        ```  
    Repalce this:  
        ```
        #ifndef itertype
        #define itertype(v) typeof((v).begin())
        #endif
        ```  
    To this:  
        ```
        #ifndef itertype
        #define itertype(v) __typeof((v).begin())
        #endif
        ```

- Go to game/DragonSoul.cpp replace this:
        float fCharge = vec_chargings[idx] * (100 + iBonus) / 100.f;
        fCharge = std::MINMAX <float> (0.f, fCharge, 100.f);
    on this:
        float fCharge = vec_chargings[idx] * (100 + iBonus) / 100.f;
        auto clip = [](float floor, float x, float ceiling)
        {
            return std::min(ceiling, std::max(floor, x));
        };
        fCharge = clip(0.f, fCharge, 100.f);

- Go to libgame/src/grid.cc add this:
        #include <algorithm>
    after this:
        #include <stdio.h>
    and replace that:
        int iSize = std::MIN(w * h, pkGrid->m_iWidth * pkGrid->m_iHeight);
    to this:
        int iSize = std::min(w * h, pkGrid->m_iWidth * pkGrid->m_iHeight);
    Go to db/grid.cpp replace this:
        int iSize = std::MIN(w * h, pkGrid->m_iWidth * pkGrid->m_iHeight);
    to this:
        int iSize = std::min(w * h, pkGrid->m_iWidth * pkGrid->m_iHeight);

- Go to game/stdafx.h add this:
        #include <random>
    after this:
        #include <algorithm>
        #include <float.h>
    Go to game/char_battle.cpp replace this (twice):
        random_shuffle(vec_bSlots.begin(), vec_bSlots.end());
    to this:
        std::random_device rd;
        std::mt19937 g(rd());
        std::shuffle(vec_bSlots.begin(), vec_bSlots.end(), g);

- Go to game/char.h repalce this:
        boost::unordered_map<VID, size_t> TargetVIDMap;
    to this:
        boost::unordered_map<DWORD, size_t> TargetVIDMap;
    Go to game/char_skill.cpp repale this:
        boost::unordered_map<VID, size_t>::iterator iterTargetMap = rSkillUseInfo.TargetVIDMap.find(TargetVID);
    to this:
        auto iterTargetMap = rSkillUseInfo.TargetVIDMap.find(TargetVID);
    Go to game/sectree.h replace this:
        LPSECTREE_LIST::iterator it_tree = m_neighbor_list.begin();
    to this:
        auto it_tree = m_neighbor_list.begin();

- Replace all "typeof" and "auto_ptr" to "__typeof" and "unique_ptr" in game and db

- Go to db/Main.cpp and remove this:
        #ifdef __FreeBSD__
        extern const char * _malloc_options;
        #endif
    and this:
        #ifdef __FreeBSD__
            _malloc_options = "A";
        #endif

- Go to db/ClientManager.h and game/p2p.h and replace this:
        #include <boost/unordered_map.hpp>
        #include <boost/unordered_set.hpp>
    to this:
        #include <unordered_map>
        #include <unordered_set>
    then, replace "boost::" to "std::". Example:
        typedef std::unordered_map<short, BYTE> TChannelStatusMap;
    Go to game/stdafx.h remove this:
        #ifdef __GNUC__
        #include <float.h>
        #include <tr1/unordered_map>
        #include <tr1/unordered_set>
        #define TR1_NS std::tr1
        #else
        #include <boost/unordered_map.hpp>
        #include <boost/unordered_set.hpp>
        #define TR1_NS boost
        #define isdigit iswdigit
        #define isspace iswspace
        #endif
    Go to game/fifo_allocator.h and remove this:
        #ifdef __GNUC__
        #include <tr1/unordered_map>
        #define TR1_NS std::tr1
        #else
        #include <boost/unordered_map.hpp>
        #define TR1_NS boost
        #endif
    Go to game/debug_allocator_adapter.h and remove this:
        #ifdef __GNUC__
        #include <tr1/unordered_map>
        #define TR1_NS std::tr1
        #else
        #include <boost/unordered_map.hpp>
        #define TR1_NS boost
        #endif
    Replace all "TR1_NS::" to "std::" in game and db

- Go to libthecore/src gost.c, tea.c, utils.c, xmd5.c and remove all "register", example:
        register DWORD n1, n2;
    should be like that:
        DWORD n1, n2;
    Go to game/matrix_card.cpp and remove all "register" too.

- Remove game/minilzo.h and game/minilzo.c
    Go to game lzo_manager.h, main.cpp, MarkImage.h, test.cpp, test_window.cpp and replace this:
        #include "minilzo.h"
    to this:
        #include <minilzo/minilzo.h>
    Then, compile new version lzo or minilzo lib.
    I will use this:
        https://github.com/CCHyper/minilzo
    Put your compiled files to new directory in extern, example:
        extern/include/minilzo/lzoconf.h
        extern/include/minilzo/lzodefs.h
        extern/include/minilzo/minilzo.h
        extern/lib/libminilzo.a

- Go to game/input_db.cpp replace this:
        CHARACTER_MANAGER::instance().for_each_pc(std::mem_fun(&CHARACTER::ComputePoints));
    to this:
        CHARACTER_MANAGER::instance().for_each_pc(std::mem_fn(&CHARACTER::ComputePoints));
    
- Go to game/messenger_manager.cpp replace this:
        DBManager::instance().FuncQuery(std::bind1st(std::mem_fun(&MessengerManager::LoadList), this),
    to this:
        DBManager::instance().FuncQuery(std::bind(&MessengerManager::LoadList, this, std::placeholders::_1),
    Go to game/mob_manager.cpp replace this:
        CHARACTER_MANAGER::instance().for_each_pc(std::bind1st(std::mem_fun(&CMobManager::RebindMobProto),this));
    to this:
        CHARACTER_MANAGER::instance().for_each_pc(std::bind(&CMobManager::RebindMobProto, this, std::placeholders::_1));
    Go to game/guild_manager.cpp replace this:
        DBManager::instance().FuncQuery(std::bind1st(std::mem_fun(&CGuild::SendGuildDataUpdateToAllMember), iter->second),
    to this:
        DBManager::instance().FuncQuery(std::bind(&CGuild::SendGuildDataUpdateToAllMember, iter->second, std::placeholders::_1),
    Go to game/guild.cpp
        DBManager::instance().FuncQuery(std::bind1st(std::mem_fun(&CGuild::LoadGuildData), this),
    to this:
        DBManager::instance().FuncQuery(std::bind(&CGuild::LoadGuildData, this, std::placeholders::_1),
    then replace this:
        DBManager::instance().FuncQuery(std::bind1st(std::mem_fun(&CGuild::LoadGuildGradeData), this),
    to this:
        DBManager::instance().FuncQuery(std::bind(&CGuild::LoadGuildGradeData, this, std::placeholders::_1),
    then replace this:
        DBManager::instance().FuncQuery(std::bind1st(std::mem_fun(&CGuild::LoadGuildMemberData), this),
    to this:
        DBManager::instance().FuncQuery(std::bind(&CGuild::LoadGuildMemberData, this, std::placeholders::_1),
    then replace this:
        DBManager::instance().FuncQuery(std::bind1st(std::mem_fun(&CGuild::__P2PUpdateGrade),this),
    to this:
        DBManager::instance().FuncQuery(std::bind(&CGuild::__P2PUpdateGrade, this, std::placeholders::_1),
    then replace this:
        DBManager::instance().FuncAfterQuery(void_bind(std::bind1st(std::mem_fun(&CGuild::RefreshCommentForce),this),ch->GetPlayerID()),
    to this:
        DBManager::instance().FuncAfterQuery(std::bind(&CGuild::RefreshCommentForce,this, ch->GetPlayerID()),
    then replace this:
        for_each(m_memberOnline.begin(), m_memberOnline.end(), std::bind1st(std::mem_fun_ref(&CGuild::SendSkillInfoPacket),*this));
    to this:
        std::for_each(m_memberOnline.begin(), m_memberOnline.end(), std::bind(&CGuild::SendSkillInfoPacket, this, std::placeholders::_1));
    then replace this:
        for_each(m_memberOnline.begin(), m_memberOnline.end(), std::bind1st(std::mem_fun_ref(&CGuild::SendSkillInfoPacket),*this));
    to this:
        for_each(m_memberOnline.begin(), m_memberOnline.end(), std::bind(&CGuild::SendSkillInfoPacket, this, std::placeholders::_1));
    then replace this:
        for_each(m_memberOnline.begin(), m_memberOnline.end(), std::bind1st(std::mem_fun(&CGuild::SendGuildInfoPacket), this));
    to this:
        for_each(m_memberOnline.begin(), m_memberOnline.end(), std::bind(&CGuild::SendGuildInfoPacket, this, std::placeholders::_1));

- Go to game/char_manager.cpp replace this:
        #ifndef __GNUC__
        #include <boost/bind.hpp>
        #endif
    to this:
        #include <boost/bind.hpp>
    then remove this:
        #ifdef __GNUC__
            using namespace __gnu_cxx;
        #endif
    then replace this:
        #ifdef __GNUC__
                    transform(m_map_pkPCChr.begin(), m_map_pkPCChr.end(), back_inserter(v), select2nd<NAME_MAP::value_type>());
        #else
                    transform(m_map_pkPCChr.begin(), m_map_pkPCChr.end(), back_inserter(v), boost::bind(&NAME_MAP::value_type::second, _1));
        #endif
    to this:
        transform(m_map_pkPCChr.begin(), m_map_pkPCChr.end(), back_inserter(v), boost::bind(&NAME_MAP::value_type::second, _1));
    then replace this:
        #ifdef __GNUC__
                    transform(m_set_pkChrState.begin(), m_set_pkChrState.end(), back_inserter(v), identity<CHARACTER_SET::value_type>());
        #else
                    v.insert(v.end(), m_set_pkChrState.begin(), m_set_pkChrState.end());
        #endif
    to this:
        v.insert(v.end(), m_set_pkChrState.begin(), m_set_pkChrState.end());
    then remove this:
        #ifdef __GNUC__
            using namespace __gnu_cxx;
        #endif
    then replace this:
        #ifdef __GNUC__
            transform(r.begin(), r.end(), back_inserter(*this), identity<CHARACTER_SET::value_type>());
        #else
            insert(end(), r.begin(), r.end());
        #endif
    to this:
        insert(end(), r.begin(), r.end());
    then replace this:
        for_each(v.begin(), v.end(), bind2nd(mem_fun(&CHARACTER::UpdateCharacter), iPulse));
    to this:
        for_each(v.begin(), v.end(), std::bind(&CHARACTER::UpdateCharacter, std::placeholders::_1, iPulse));
    then replace this:
        for_each(v.begin(), v.end(), bind2nd(mem_fun(&CHARACTER::UpdateStateMachine), iPulse));
    to this:
        for_each(v.begin(), v.end(), std::bind(&CHARACTER::UpdateStateMachine, std::placeholders::_1, iPulse));
    then replace this:
        for_each(i.begin(), i.end(),
                bind2nd(mem_fun(&CHARACTER::UpdateStateMachine), iPulse));
    to this:
        for_each(i.begin(), i.end(), std::bind(&CHARACTER::UpdateStateMachine, std::placeholders::_1, iPulse));

- Go to game/config.cpp replace this (three times):
        if (NULL != line[0])
    to this:
        if ('\0' != line[0])

- Go to game/char_skill.cpp replace this:
        if (false == 
        m_SkillUseInfo[dwVnum].UseSkill(
            bUseGrandMaster,
            (NULL != pkVictim && SKILL_HORSE_WILDATTACK != dwVnum) ? pkVictim->GetVID() : NULL,
            ComputeCooltime(iCooltime * 1000),
            iSplashCount,
            lMaxHit))
    to this:
        if (!m_SkillUseInfo[dwVnum].UseSkill(bUseGrandMaster, (NULL != pkVictim and SKILL_HORSE_WILDATTACK != dwVnum) ? pkVictim->GetVID() : 0, ComputeCooltime(iCooltime * 1000), iSplashCount, lMaxHit))

- Go to game/cmd_gm.cpp replace this:
        if (*szName == NULL || *szChangeAmount == '\0')
    to this:
        if (*szName == '\0' or * szChangeAmount == '\0')

- Go to game/char_item.cpp replace this (five times):
        std::vector <LPITEM> item_gets(NULL);
    to this:
        std::vector <LPITEM> item_gets(0);

- Go to game/utils.cpp replace this:
        if (NULL == w[1])
    to this:
        if (!w[1])
    then replace:
        if (NULL == *s)
    to this:
        if (!*s)
    then replace:
        if (NULL == *w)
    to this:
        if (!*w)

- Go to game/questlua_pc.cpp replace this:
        std::vector <LPITEM> item_gets(NULL);
    to this:
        std::vector <LPITEM> item_gets(0);

- Go to game/sectree_manager.cpp replace this:
        unsigned int uiSize;
        unsigned int uiDestSize;
    to this:
        long unsigned int uiSize;
        long unsigned int uiDestSize;