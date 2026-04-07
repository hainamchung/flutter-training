# Verify — Full Capstone Project

> 📌 **Hoàn thành [03-exercise.md](./03-exercise.md) trước khi làm verify.**

---

## Evaluation Rubric

### Scoring Breakdown (100 points)

| Tiêu chí | Trọng số | Points |
|-----------|----------|--------|
| **Architecture** | 25% | 25 |
| **Functionality** | 20% | 20 |
| **Code Quality** | 20% | 20 |
| **Testing** | 20% | 20 |
| **Production Readiness** | 15% | 15 |
| **Total** | 100% | 100 |

---

## Architecture Checklist (25 points)

| # | Criteria | Points | Check |
|---|----------|--------|-------|
| 1.1 | Clean architecture layers (Presentation → Domain → Data) | 5 | [ ] |
| 1.2 | Correct dependency direction (inner to outer) | 5 | [ ] |
| 1.3 | Feature-first folder structure | 5 | [ ] |
| 1.4 | Repository pattern (abstraction) | 5 | [ ] |
| 1.5 | State management follows base_flutter conventions | 5 | [ ] |

### Architecture Quality Indicators

```
✅ PASS (20-25 pts):
- Clear layer separation
- No circular dependencies
- Repository interfaces defined

⚠️ NEEDS IMPROVEMENT (10-19 pts):
- Some layer mixing
- 1-2 circular dependencies

❌ FAIL (0-9 pts):
- All code in one layer
- No repository pattern
```

---

## Functionality Checklist (20 points)

| # | Criteria | Points | Check |
|---|----------|--------|-------|
| 2.1 | Profile view: loading state (shimmer) | 3 | [ ] |
| 2.2 | Profile view: data state (all fields) | 4 | [ ] |
| 2.3 | Profile view: error state (retry) | 3 | [ ] |
| 2.4 | Edit profile: form validation | 4 | [ ] |
| 2.5 | Edit profile: avatar upload (camera/gallery) | 4 | [ ] |
| 2.6 | Edit profile: save & refresh | 2 | [ ] |

### Functionality Quality Indicators

```
✅ PASS (16-20 pts):
- All features work end-to-end
- Edge cases handled
- Error states shown

⚠️ NEEDS IMPROVEMENT (10-15 pts):
- Basic features work
- Some edge cases missing

❌ FAIL (0-9 pts):
- Features broken
- No error handling
```

---

## Code Quality Checklist (20 points)

| # | Criteria | Points | Check |
|---|----------|--------|-------|
| 3.1 | No hardcoded strings (i18n) | 4 | [ ] |
| 3.2 | No hardcoded colors (AppTheme) | 4 | [ ] |
| 3.3 | Proper null safety usage | 4 | [ ] |
| 3.4 | Readable naming conventions | 4 | [ ] |
| 3.5 | Proper error handling (try/catch) | 4 | [ ] |

### Code Quality Indicators

```
✅ PASS (16-20 pts):
- Clean code, follows conventions
- Proper error handling
- No code smells

⚠️ NEEDS IMPROVEMENT (10-15 pts):
- Minor issues
- Some hardcoded values

❌ FAIL (0-9 pts):
- Hardcoded throughout
- No error handling
```

---

## Testing Checklist (20 points)

| # | Criteria | Points | Check |
|---|----------|--------|-------|
| 4.1 | Unit test coverage ≥ 70% | 8 | [ ] |
| 4.2 | Repository tests | 4 | [ ] |
| 4.3 | ViewModel tests | 4 | [ ] |
| 4.4 | Widget tests (all states) | 2 | [ ] |
| 4.5 | Golden tests (light/dark) | 2 | [ ] |

### Testing Quality Indicators

```
✅ PASS (16-20 pts):
- Coverage ≥ 70%
- All tests pass
- Meaningful assertions

⚠️ NEEDS IMPROVEMENT (10-15 pts):
- Coverage 50-69%
- Some flaky tests

❌ FAIL (0-9 pts):
- Coverage < 50%
- Tests failing
```

---

## Production Readiness Checklist (15 points)

| # | Criteria | Points | Check |
|---|----------|--------|-------|
| 5.1 | CI pipeline passes (lint + test + build) | 5 | [ ] |
| 5.2 | Error handling (network, validation, unknown) | 4 | [ ] |
| 5.3 | Image caching & optimization | 3 | [ ] |
| 5.4 | Unsaved changes warning | 3 | [ ] |

### Production Readiness Indicators

```
✅ PASS (12-15 pts):
- CI green
- Proper error handling
- Performance optimized

⚠️ NEEDS IMPROVEMENT (7-11 pts):
- CI mostly green
- Some edge cases missing

❌ FAIL (0-6 pts):
- CI failing
- No error handling
```

---

## Final Score Calculation

### Fill in your scores:

| Section | Max | Your Score |
|---------|-----|------------|
| Architecture | 25 | ___ |
| Functionality | 20 | ___ |
| Code Quality | 20 | ___ |
| Testing | 20 | ___ |
| Production Readiness | 15 | ___ |
| **TOTAL** | **100** | **___** |

### Score Interpretation

| Score | Level | Description |
|-------|-------|-------------|
| **90-100** | **Excellent** 🌟 | Vượt expectations, production-ready |
| **75-89** | **Pass** ✅ | Đạt chuẩn middle-level |
| **60-74** | **Conditional Pass** 🟡 | Cần fix trong 3 ngày |
| **< 60** | **Fail** 🔴 | Cần thêm training time |

---

## Code Walkthrough Questions

### Architecture (5 min)

1. Tại sao chọn folder structure này?
2. Dependency flow như thế nào?
3. Repository pattern hoạt động ra sao?

### State Management (5 min)

1. Chọn Riverpod vì lý do gì?
2. AsyncValue pattern hoạt động thế nào?
3. Form state được quản lý ra sao?

### Testing (5 min)

1. Coverage report hiển thị gì?
2. Unit tests cover những cases nào?
3. Golden tests verify những gì?

### Demo Flow (10 min)

1. **Happy Path**: View profile → Edit → Save → Verify
2. **Error Path**: Network error → Retry → Success
3. **Avatar Upload**: Camera → Crop → Upload → Verify
4. **Settings**: Toggle dark mode → Verify
5. **Logout**: Confirm → Clear → Navigate

---

## Demo Checklist

### Pre-Demo

- [ ] App running on device
- [ ] Mock data working
- [ ] All tests passing locally
- [ ] CI pipeline green
- [ ] Screenshots prepared

### Demo Flow

- [ ] Show profile view (loading, data, error states)
- [ ] Show edit profile form
- [ ] Show avatar upload flow
- [ ] Show settings toggles
- [ ] Show logout flow
- [ ] Show test coverage
- [ ] Show CI pipeline

### Post-Demo

- [ ] Answer questions
- [ ] Show code structure
- [ ] Explain decisions

---

## Feedback & Improvement

### What went well?

```
1.
2.
3.
```

### What could be improved?

```
1.
2.
3.
```

### Key learnings?

```
1.
2.
3.
```

---

## Next Steps

### For Pass (75+ points)

✅ Chúc mừng! Bạn đã đạt chuẩn middle-level Flutter developer.
- Continue với production work
- Mentor junior developers
- Explore advanced topics

### For Conditional Pass (60-74 points)

🟡 Cần fix trong 3 ngày:
- Review feedback từ reviewer
- Fix identified issues
- Resubmit for re-evaluation

### For Fail (< 60 points)

🔴 Cần thêm training time:
- Meet với facilitator
- Create improvement plan
- Set new timeline

---

## Resources

- **Base Flutter:** `../../base_flutter/`
- **Rubric:** `../tieu-chuan/middle-level-rubric.md`
- **AI Toolkit:** `../ai-toolkit/ai-driven-development.md`
- **Prompt Dojo:** `../ai-toolkit/prompt-dojo.md`

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
